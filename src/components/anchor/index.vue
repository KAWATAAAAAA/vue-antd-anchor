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
          "[object Array]": () => {}
        };
        // 此处 || 永远会执行左值
        //return ACTIONS[Object.prototype.toString.call(this.lazy)]() || this._bornChildComponent;
        return ACTIONS[Object.prototype.toString.call(this.lazyStore)]();
      }
    }
  },
  data() {
    return {
      blockList: [],
      /* 生效样式列表 */
      commonClassList: [],
      /* 用于节流 scroll 事件 */
      scrollTimer: "",
      /* 标识滚动事件是否完成 */
      isScrollDone: false,
      lazyStore: ""
    };
  },
  watch: {
    /* 当做钩子函数，在值发生改变后做些什么 */
    isScrollDone(newval) {
      if (newval) {
        console.log("scrolled");
      } else {
        console.log("scrolling");
      }
    }
  },
  created() {
    this._cloneDeepPropLazy();
  },
  mounted() {
    /* 挂载完毕，表示模板中的 v-for已经完成渲染 */

    /* 模板渲染完成后[必须]，侵入传入的组件，加上预设class，注入占位input */
    this.initDefaultSoltComponents();

    /* 模板渲染完成后[必须]，侵入完成v-for渲染的label标签,加上for*/
    this.initCreatedLabel();

    this.initClassConfig();
    //console.log(this.blockList[0].tag)

    /* 监听本组件顶级 wrapper */
    this.initListenerWrapper();
  },
  methods: {
    initDefaultSoltComponents() {
      console.log(this.$slots.default);

      this.blockList = this.$slots.default;

      /* 近似危险的操作，在该组件中改变了传入组件的属性  [ 而非状态 ] */
      /* 如果子 VNode 中存在 elm 说明元素已经渲染到页面，执行初始化注入 */
      this.$slots.default
        .filter(item => item.elm)
        .map(item => {
          /* 默认传入的组件添加样式 */
          item.elm.classList.add("block-wrapper");

          /* 默认传入的组件中注入 input 标签  */
          let input = document.createElement("input");
          /* 默认使用 vue tag作为占位 input 的 id */
          input.setAttribute("id", item.tag);
          input.classList.add("invisible-input-placeholder");

          item.elm.appendChild(input);
        });
    },
    initCreatedLabel() {
      /* 取代在 v-for 中直接使用 :for="" ,因为此时 mounted 还未注入 tag */

      let labelList = document.querySelectorAll(
        ".nav-line > [data-unique='nav-label']"
      );

      this.blockList.map((item, index) => {
        labelList[index].setAttribute("for", item.tag);
      });
    },
    initClassConfig() {
      /* 指定本组件wrapper 是否允许溢出 */
      this.scrollView ? this.commonClassList.push("scroll-view") : undefined;
    },
    initListenerWrapper() {
      this.$refs["scroll-wrapper"].addEventListener("scroll", () => {
        /*  scroll 节流，完成后传递完成状态 */
        this.isScrollDone = false;
        clearTimeout(this.scrollTimer);
        this.scrollTimer = setTimeout(() => {
          this.isScrollDone = true;
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
      //active_index
      /* 遍历 default slot 中的子组件，提取出出生状态的VNode */
        console.log(active_index.target.getAttribute("key"))
        
        //console.log(event)
      let born = [];
      this.$slots.default.forEach((item, index) => {
        item.elm ? born.push(index) : undefined;
      });
    
      //console.log(born);
    }

  },
  render(h) {
    //let _bornChildComponent = new Set();
    
    const _injectLabel = () => {
        console.log(this._self)
        let _that = this._self
      return Array.apply(null, { length: _that.labelName.length }).map(
        (item, index) => {
          return h(
            "label",
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
        //...this.$slots.default, // 子节点槽位
        ...this._childComponentLazyLoadMarkInitFactory,
        h("div", { attrs: { class: "nav-line" } }, _injectLabel())
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
::v-deep .block-wrapper > .invisible-input-placeholder {
  position: absolute;
  top: 0;
  height: 100%;
  width: 1px;
  border: 0;
  padding: 0;
  margin: 0;
  clip: rect(0 0 0 0);
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

  > label {
    &:hover {
      cursor: pointer;
    }
    width: 100%;
  }
}
</style>
