<template>
  <div
    class="my-pull-refresh"
    @touchstart="handleTouchStart"
    @touchmove="handleTouchMove"
    @touchend="handleTouchEnd"
  >
    <div
      class="refresh-wrapper"
      :style="{
        height: refreshHeight + 'px',
        backgroundColor: bgColor,
        transition: transitionStatus
      }"
      v-show="refreshShow"
    >
      <span
        :style="{
          color: tipColor,
          fontSIze: tipSize + 'px',
        }"
        >{{ refreshTip }}</span
      >
    </div>
    <div class="content-wrapper">
      <slot></slot>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, toRefs, computed } from "vue";
import { IProps, IData, ITouchingPosition, DefaultTips, DefaultConfigs } from "./types";

// 当loading状态不能拖动
let loadingStatus = false

const emit = defineEmits<{
  (e: "refreshing"): void;
  (e: "refreshed"): void;
}>();

const props = withDefaults(defineProps<IProps>(), {
  willPullTip: DefaultTips.WILL_PULL_TIP,
  pullingTip: DefaultTips.PULLING_TIP,
  loadingTip: DefaultTips.LOADING_TIP,
  tipColor: DefaultConfigs.TIP_COLOR,
  tipSize: DefaultConfigs.TIP_SIZE,
  bgColor: DefaultConfigs.BG_COLOR,
  loadingDuration: DefaultConfigs.LOADING_DURATION,
});

const data = reactive<IData>({
  refreshTip: props.willPullTip,
  refreshHeight: 0,
  refreshShow: false,
  needTransition: false
});
const { refreshTip, refreshHeight, refreshShow } = toRefs(data);
let touchingPosition: ITouchingPosition = {
    start: 0,
    end: 0
}

const handleTouchStart = (e: TouchEvent) => {
    console.log('start---', e)
    const touch = e.changedTouches[0]
    touchingPosition.start = touch.clientY
}
const handleTouchMove = (e: TouchEvent) => {
    console.log('move---', e)
    if (!loadingStatus) {
        setRefreshShow(true)
        const touch = e.changedTouches[0]
        touchingPosition.end = touch.clientY
        const distance = touchingPosition.end - touchingPosition.start
        if (data.refreshHeight > DefaultConfigs.MIN_REFRESHING_HEIGHT) {
            setTip(props.pullingTip)
        }
        // 判断向下还是向上拉
        if (touchingPosition.start > touchingPosition.end) { 
            // 手指从下向上
            addRefreshHeight(distance)
        } else {
            addRefreshHeight(distance / 2)
        }
        // 每次move的时候，把end赋值给start，其实就是每一次增加的距离 / 2
        touchingPosition.start = touchingPosition.end
    }  
}
const handleTouchEnd = (e: TouchEvent) => {
    console.log('end---', e)
    setNeedTransition(true)
    if (data.refreshHeight < DefaultConfigs.MIN_REFRESHING_HEIGHT) {
        resetRefresh()
    } else {
        setRefreshing()
        setTimeout(setRefreshed, props.loadingDuration)
    }
}
const resetRefresh = () => {
    setRefreshHeight(0)
    setTip(props.willPullTip)
    setLoadingStatus(false)
    closeRefreshArea()
}
const setRefreshing = () => {
    setRefreshHeight(DefaultConfigs.MIN_REFRESHING_HEIGHT)
    setTip(props.loadingTip)
    setLoadingStatus(true)
    emit('refreshing')
}
const setRefreshed = () => {
    resetRefresh()
    emit('refreshed')
}
// 改变文字状态
const setTip = (tip: string) => {
    data.refreshTip = tip
}
const setRefreshHeight = (height: number) => {
    data.refreshHeight = height
}
const addRefreshHeight = (distance: number) => {
    data.refreshHeight += distance
}
const setNeedTransition = (status: boolean) => {
    data.needTransition = status
}
const setRefreshShow = (status: boolean) => {
    data.refreshShow = status
}
const setLoadingStatus = (status: boolean) => {
    loadingStatus = status 
}
const resetRefreshHeight = () => {
    setRefreshHeight(0)
    setTip(props.willPullTip)
    setLoadingStatus(false)
    closeRefreshArea()
}
const closeRefreshArea = () => {
    setTimeout(() => {
        setNeedTransition(false)
        setRefreshShow(false)
    }, DefaultConfigs.TRANSITION_DURATION * 1000)
}

const transitionStatus = computed(() => 
    data.needTransition ? `height ${DefaultConfigs.TRANSITION_DURATION}s` : 'none '
)
</script>

<style lang="scss">
.my-pull-refresh {
  overflow-y: auto;
  height: 100vh;
  .refresh-wrapper {
    display: flex;
    justify-content: center;
    align-items: center;
    overflow: hidden;
  }
}
</style>

