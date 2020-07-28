<script>
export default {
  name: "anchor",
  props: {
    scrollView: {
      required: false,
      type: Boolean,
      default: () => {
        return false;
      }
    },
    labelName: {
      required: true,
      type: Array,
      validator: valueList => {
        let setList = new Set();
        valueList.forEach(item => {
          setList.add(item);
        });
        if (valueList.length !== setList.size) {
          throw new Error(
            "Please check the 'labelName' [Array]， Like a [Object Set] features"
          );
        } else {
          return true;
        }
      }
    },
    lazy: {
      required: false,
      type: [Boolean, String, Array],
      default: () => {
        return false;
      },
      validator: value => {
        if (
          (typeof value === "string" && value === "lazy") ||
          typeof value === "boolean" ||
          (Array.isArray(value) &&
            value.every(item => typeof item === "number"))
        ) {
          return true;
        } else {
          throw new TypeError(
            "Please check the 'lazy' [Boolean] or [String('lazy')] or [Array(index : Number)]"
          );
        }
      }
    }
  },
  computed: {
    _childComponentLazyLoadMarkInitFactory: {
      get: function() {
        /**
         * @懒加载规则
         *
         * @param [Boolean]
         * 若传入的值为 布尔值且为 true，则自动启用懒加载，默认第一个元素显示，其余的懒加载
         *
         * @param [String]
         * 若传入的值为 String 则根据布尔值的逻辑执行
         *
         * @param [Array]
         * 若传入的值为 Array 则遍历该数组将对应下标的子组件置为 lazy load
         */
        let _that = this
        const ACTIONS = {
          "[object Boolean]": () => {
            if (this.lazyStore) {
              /* 出生记录 */
              _that.bornChildComponent.push(0)
              return [this.$slots.default[0]];
            } else {
              _that.bornChildComponent = new Array(_that.$slots.default.length).fill(0).map((item, index) => index)
              return _that.$slots.default;
            }
          },
          "[object Array]": () => {
            return _that.$slots.default.filter((item,index) => _that.lazyStore.includes(index) )
          }
        };
        // 此处 || 永远会执行左值,vue 不会触发计算属性
        //return ACTIONS[Object.prototype.toString.call(this.lazy)]() || this.bornChildComponent;

        /* 逗号表达式，能使表达式两边语句都执行，并且最终返回右值 */
        return  this.bornChildComponent.length,ACTIONS[Object.prototype.toString.call(this.lazyStore)]();
      }
    }
  },
  data() {
    return {

      /* 生效样式列表 */
      commonClassList: [],
      /* 用于节流 scroll 事件 */
      scrollTimer: "",
      /* 标识滚动事件是否完成 ,false 为正在滚动，true为滚动完成*/
      isScrollDone: "",
      /* 深拷贝Lazy中间件，用以触发 computed属性 */
      lazyStore: "",
      /* 出生证明 */
      bornChildComponent: [],

      /* ink小圆点中间变量 */
      lastActiveIndex: 1,
      currentActiveIndex: 1,

      /* xOb */
      xOb:""
    };
  },
  watch: {
    /* 当做钩子函数，在值发生改变后做些什么 */
    isScrollDone(newval) {
      newval?this.$emit("onScrolled"):this.$emit("onScrolling")
    },
    bornChildComponent(){
      this.bornChildComponent.length === this.$slots.default.length?this.$emit("onAllDone"):undefined
    
    }
  },
  created() {
    /* render函数未执行之前，先将传过来的 Lazy执行深克隆 */
    this._cloneDeepPropLazy();
  },

  updated(){
    this.initAddIntersectionListener()
  },
  mounted() {
    /* 挂载完毕，表示模板中的 v-for已经完成渲染 */

    /* 组件外层wrapper 设定默认样式 */
    this.initClassConfig();

    /* 监听本组件顶级 wrapper */
    this.initListenerWrapper();

    /* 交叉观察者监听页面元素是否进入viewport */
    this.initAddIntersectionListener()

  },
  methods: {

    initClassConfig() {
      /* 指定本组件wrapper 是否允许溢出 */
      this.scrollView ? this.commonClassList.push("scroll-view") : undefined;
    },
    initListenerWrapper() {
      let _that = this
      this.$refs["scroll-wrapper"].addEventListener("scroll", () => {
        /*  scroll 节流，完成后传递完成状态 */
        _that.isScrollDone = false;
        clearTimeout(_that.scrollTimer);
        _that.scrollTimer = setTimeout(() => {
          _that.isScrollDone = true;
        }, 100);
      });
    },
    initAddIntersectionListener(){
      this.xOb?this.xOb.disconnect():undefined

      this.xOb = new IntersectionObserver( IntersectionList => {
        //console.log("正在接受监听的元素长度" + IntersectionList.length)
            
              let inView = IntersectionList.filter(item => item.boundingClientRect.top === 0)
              console.log(inView)
            },{
              root:document.querySelectorAll(".full-vw-vh")[0],

            })
            
            let elms = []

            this.$slots.default.forEach( vnode => {
              if(vnode.elm){
                elms.push(vnode.elm)
              }
            })
            console.log(elms.length + "个元素被渲染了")
            elms.forEach(elm => {
                this.xOb.observe(elm)
            })
    },
    _cloneDeepPropLazy() {
      const ACTIONS = {
        "[object Boolean]": () => {
          this.lazyStore = JSON.parse(JSON.stringify(this.lazy));
        },

        "[object Array]": () => {
          let newArr = [];
          this.lazy.forEach(item => {
            newArr.push(item);
          });

          this.lazyStore = newArr;
          this.bornChildComponent = JSON.parse(JSON.stringify(newArr))
        }
      };
      ACTIONS[Object.prototype.toString.call(this.lazy)]();
    },
    _wakeUpLazyLoadComponent(active_index) {
      /* 遍历 default slot 中的子组件，提取出出生状态的VNode */

      /* 若非懒加载模式，则退出该函数 */
      if(!this.lazy){
        return
      }
      /* 由于各种原因，导致入参为 Event 事件，就只好利用这种办法来获取点击到的 index */
      /* render 函数中已提供 key 属性 */
      let currentIndex = parseInt(active_index.target.getAttribute("key"));
    
      /* 出生证明 */
      this.bornChildComponent.push(currentIndex)

      /* 跨类型修改 传入的拷贝 lazy */
      if(typeof this.lazyStore === "boolean"){
        this.lazyStore = [0,currentIndex]
      } else if(Array.isArray(this.lazyStore)){
        this.lazyStore.push(currentIndex)
      }
    
      /* 预计在此提供一个钩子函数 */
    },
    _scrollToElmView(active_index){
      let currentIndex = parseInt(active_index.target.getAttribute("key"));
       this.$slots.default[currentIndex].elm.scrollIntoView({
        behavior: "smooth"
      })
      /* 改变ink 位置 */
      this.$refs["tiny-ink"].style.transform = `translateX(-50%) translateY(calc(${currentIndex + 1} * 34px - 50%))`
    
      /* 点击事件钩子 */
      this.$emit("onClick",this.$slots.default[currentIndex].child,currentIndex)
    }

  },
  render(h) {
  
    const _injectLabel = () => {
        
      let _that = this._self
      return Array.apply(null, { length: _that.labelName.length }).map(
        (item, index) => {
          return h(
            "li",
            {
              attrs: {
                "data-unique": "nav-label",
                /* 用来做渲染后的标识 */
                key: index,
              },
              key: index,
              on:{
                  "~click": (index) => {     
                    _that._wakeUpLazyLoadComponent(index)
                  },
                  "click":(index) => {
                    _that._scrollToElmView(index)
                  }
                }
            },
            [_that.labelName[index]]
          );
        }
      );
    };

  

    return h(
      "div", // 组件 wrapper
      {
        //class:this.commonClassList,
        attrs: {
          class: `full-vw-vh ${this.commonClassList
            .toString()
            .replace(/,/g, " ")}`
        },
        ref: "scroll-wrapper"
      },
      [
        /* ...this.$slots.default 子节点槽位，直接解构则不是懒加载 */
        ...this._childComponentLazyLoadMarkInitFactory,
        h("ul", { attrs: { class: "nav-line" } },
        [
          h("span",{attrs:{class:"tiny-ink"},ref: "tiny-ink"}),
           ..._injectLabel()
        ]
        )
      ]
    );
  }
};
</script>
<style lang="scss" scoped>
.full-vw-vh {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  scroll-behavior: smooth;

  &::-webkit-scrollbar   
  {  
    display: none;
  }  
/*   &::-webkit-scrollbar-track {
    background:transparent;
  }
  &::-webkit-scrollbar-thumb{
    border-radius: 10px;
    background-color:#95afc0;
  } */
}
::v-deep .block-wrapper {
  /* 组件默认高度 根据组件本身定义的高度来扩展 */
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}

.scroll-view {
  overflow: auto;
}
.nav-line {

  width: 200px;
  height: auto;
  padding: 1rem 0;
  position: fixed;
  right: 0;
  top: 20px;

  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;

  > li {
    display: block;
    padding: .5rem 1.5rem;
    height: auto;
    text-align: left;
    word-wrap:break-word;
    &:hover {
      cursor: pointer;
    }
    width: 100%;
  }
  &::before{
    content: '';
    width: 2px;
    height: 100%;
    background: #e8e8e8;
    position: absolute;
    top: 0;
    left: 0;
    transform: translateX(-50%) ;
  }
  .tiny-ink{
    width: 1rem;
    height: 1rem;
    position: absolute;
    border: 5px solid #1890ff;
    border-radius: 50%;
    background: white;
    left: 0;
    top: 0;
    transform: translateX(-50%) translateY(calc(34px / 2 + 50%));
    transition: all .5s ease;
  
  }
}
</style>
