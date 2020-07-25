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
              //_bornChildComponent.add(this.$slots.default[0])
              return [this.$slots.default[0]];
            } else {
              return this.$slots.default;
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
      bornChildComponent: []
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
  mounted() {
    /* 挂载完毕，表示模板中的 v-for已经完成渲染 */

    /* 组件外层wrapper 设定默认样式 */
    this.initClassConfig();

    /* 监听本组件顶级 wrapper */
    this.initListenerWrapper();

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
        h("ul", { attrs: { class: "nav-line" } }, _injectLabel())
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

  width: 100px;
  height: 100vh;

  position: fixed;
  right: 0;
  top: 0;
  border: 1px solid red;

  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;

  > li {
    display: block;
    &:hover {
      cursor: pointer;
    }
    width: 100%;
  }
}
</style>
