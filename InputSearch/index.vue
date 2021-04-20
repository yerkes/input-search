<template>
  <div v-clickoutside="handleClose" class="cont">
    <el-col>
      <el-input ref="getValue" v-model="value" style="width:250px" :placeholder="placeholder" @focus="onFocus()" @blur="onBlur" @change="onChange" @input="onInput" />
      <el-button style="margin-left: 1px" type="primary" @click="search">搜索</el-button>
    </el-col>
    <el-col v-if="isSearch" class="history">
      <div />
      <div>历史记录</div>
      <ul>
        <li v-for="(item,index) in historyList" :key="index">
          <span @click="clickHistory(item,index)"><el-link :underline="false">{{ item }}</el-link></span>
          <i class="el-icon-circle-close" @click="delHistory(item,index)" />
        </li>
      </ul>
      <div class="clear" @click="clear"><el-link :underline="false">清空记录</el-link></div>
    </el-col>
    <el-col v-if="isInput" class="history">
      <div />
      <div>商家编码</div>
      <ul>
        <li v-for="(item,index) in list" :key="index">
          <span @click="clickHistory(item,index)"><el-link :underline="false">{{ item }}</el-link></span>
          <i />
        </li>
      </ul>
    </el-col>
  </div>
</template>

<script>

// 鼠标点击事件 组件外区域 有效
const clickoutside = {
  // 初始化指令
  bind(el, binding, vnode) {
    function documentHandler(e) {
      // 这里判断点击的元素是否是本身，是本身，则返回
      if (el.contains(e.target)) {
        return false
      }
      // 判断指令中是否绑定了函数
      if (binding.expression) {
        // 如果绑定了函数 则调用那个函数，此处binding.value就是handleClose方法
        binding.value(e)
      }
    }
    // 给当前元素绑定个私有变量，方便在unbind中可以解除事件监听
    el.__vueClickOutside__ = documentHandler
    document.addEventListener('click', documentHandler)
  },
  update() { },
  unbind(el, binding) {
    // 解除事件监听
    document.removeEventListener('click', el.__vueClickOutside__)
    delete el.__vueClickOutside__
  }
}

export default {
  /** 在vue里注册构造的函数**/
  name: 'SearchBar',
  directives: { clickoutside },
  props: {
    /**
     * storageName 本地缓存名称
     * list 输入激活 提示列表
     * placeholder 输入框占位符
     * isOutSelect 是否有外部选择框 操作
     */
    storageName: {
      type: String,
      default: ''
    },
    list: {
      type: Array,
      default: () => {}
    },
    placeholder: {
      type: String,
      default: '请输入内容'
    },
    isOutSelect: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      value: null,
      isInput: false,
      isSearch: false,
      inputText: '',
      historyList: [],
      isDel: false
    }
  },
  watch: {
    list(val) {
      if (val.length > 0) this.isInput = true
    },
    isOutSelect(val) {
      if (val) this.value = null
    }
  },

  // computed: {
  //   value() {
  //     return this.value
  //   }
  // },

  mounted() {
    // const el = document.getElementsByClassName('history')
    // el.addEventListener('click', () => {
    //   this.isSearch = true
    // })
    // window.addEventListener('click', () => {
    //   console.log('点击')
    //   this.outClick ? this.isSearch = false : ''
    //   this.outClick = false
    // })
  },
  methods: {
    handleClose() {
      // console.log('指令')
      if (!this.isDel) this.isSearch = false
    },

    onInput(e) {
      if (e !== '') {
        this.isSearch = false
        !this.isInput ? this.$emit('query', e) : ''
      } else {
        this.isInput = false
      }
    },

    onChange(e) {
      if (this.$refs.getValue.value !== '') {
        // console.log('enter键')
        setTimeout(() => {
          this.search()
        }, 200)
      }
    },
    // 失去焦点时触发
    onBlur() {
      setTimeout(() => { // 失去焦点后 优先级最高 设200ms后执行
        if (this.isInput || this.isDel || this.isSearch) return
        if (this.value === '') this.$emit('emery')
        this.isSearch = false
        this.isInput = false
      }, 200)
    },
    // 点击搜索框触发
    onFocus() {
      // historyList有值才执行接下来的操作
      if (localStorage.getItem(this.storageName) === null) return
      this.historyList = JSON.parse(localStorage.getItem(this.storageName))
      if (this.historyList.length !== 0) this.isSearch = true // 有值显示历史记录
    },

    // 点击历史记录直接搜索
    clickHistory(item, index) {
      this.$emit('getValue', item)
      this.isSearch = false
      this.isInput = false
      this.value = item

      this.historyList.unshift(item)

      const list = (arr) => {
        return arr.filter(function(item, index, arr) {
          return arr.indexOf(item, 0) === index
        })
      }

      this.historyList = list(this.historyList)

      localStorage.setItem(this.storageName, JSON.stringify(this.historyList)) // 新数组储存
    },

    // 删除单个历史记录
    delHistory(item, index) {
      this.isDel = true // 防止 onBlur先执行
      console.log(1)
      setTimeout(() => {
        this.isDel = false
      }, 300)
      this.historyList.splice(index, 1)
      localStorage.setItem(this.storageName, JSON.stringify(this.historyList)) // 新数组储存
      if (this.historyList.length === 0) this.isSearch = false // 删除最后一个记录
    },

    // 点击搜索按钮
    search() {
      this.isSearch = false // 每次点击搜索后历史记录就隐藏
      this.isInput = false

      if (this.$refs.getValue.value !== '') { // 判断输入框的值
        this.$emit('getValue', this.$refs.getValue.value)

        this.historyList.unshift(this.$refs.getValue.value) // 每次搜索的值添加到数组头部

        const list = (arr) => { // 数组去重
          return arr.filter(function(item, index, arr) {
            return arr.indexOf(item, 0) === index
          })
        }

        this.historyList = list(this.historyList)

        this.historyList.length > 10 ? this.historyList.slice(0, 10) : this.historyList // 最多保存10条

        localStorage.setItem(this.storageName, JSON.stringify(this.historyList)) // 存localStorage
      }
    },

    // 清空本地缓存
    clear() {
      localStorage.removeItem(this.storageName)
      this.isSearch = false
    }
  }
}
</script>

<style lang="scss" scoped>
.cont{
  display: inline-flex;
  flex-direction: column;
  position: relative;
}
$width: 230px;
$border: 1px solid #409EFF;
.history{
  background: #fff;
  margin-top: 32px;
  position: absolute;
  z-index: 999;
  width: 250px;
  padding: 0 10px;
  border:{
    bottom:$border;
    left: $border;
    right: $border;
  }
  border-bottom-left-radius: 3px;
  border-bottom-right-radius: 3px;
  &>div:nth-child(1){
    width: 230px;
    border-top: 1px solid #ddd;
  }
  &>div:nth-child(2){
    font: {
      size: 14px;
    };
    width: $width;
    margin: 15px 0;
  }
  .clear{
    float: right;
    font-size: 12px;
    color: #909399;
    margin-bottom: 10px;
  }
  ul{
    margin: 0;
    padding: 0;
    width: $width;
    color: #606266;
    font:{
      size: 12px;
    }
    list-style: none;
    li{
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 15px;
      span{
        width: 100%;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
      }
    }
  }
}
</style>
