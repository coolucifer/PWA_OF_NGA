<template>
  <div id="app">
    <!-- <transition name="page-move"> -->
      <keep-alive key="keep-alive">
        <router-view v-if="$route.meta.keepAlive" :key="new Date().getTime()"/>
      </keep-alive>
    <!-- </transition> -->
    <transition name="page-move">
      <router-view v-if="!$route.meta.keepAlive" :key="new Date().getTime()"/>
    </transition>
  </div>
</template>

<style lang="scss">
body {
  /* Disables pull-to-refresh but allows overscroll glow effects. */
  overscroll-behavior-y: none;
}
#app {
  box-sizing: border-box;
  height: 100vh;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  .page-move-enter,
  .page-move-leave-active {
    transform: translate(100%, 0);
  }

  .page-move-enter-active,
  .page-move-leave-active {
    transition: transform 0.3s;
  }

  // 各种颜色
  .royalblue {
    color: royalblue;
  }
  .darkblue {
    color: darkblue;
  }
  .blue {
    color: #0080ff;
  }
  .orange {
    color: #a06700;
  }
  .red {
    color: #da2c2c;
  }
  .green {
    color: #3d9f0e;
  }
  .bold {
    font-weight: bold;
  }
  .italic {
    font-style: italic;
  }

  // 页面加载动画颜色
  .mu-refresh-control {
    color: #EFB973;
  }
  .mu-circle-spinner {
    border-color: #EFB973;
  }
}
</style>
