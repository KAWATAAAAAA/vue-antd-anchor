<template>
  <div class="full-vw-vh" :class="commonClassList" ref="scroll-wrapper">
    <!-- 默认接收所有非具名组件 -->
    <slot></slot>
    <!-- 后期扩展右侧nav line -->
    <div class="nav-line">
      <label
        v-for="(item, index) in labelName"
        :key="index"
        data-unique="nav-label"
        >{{ item }}</label
      >
    </div>
  </div>
</template>
