<template>
  <div class="ax-swiper" ref="swiper">
    <div class="ax-swiper-wrapper" ref="wrapper"
         :class="vertical === true ? 'ax-swiper-wrapper-vertical' : ''"
    >
      <template class="ax-swiper-item" ref="firstItem" v-if="circular" v-html="firstItem"></template>
      <slot></slot>
    </div>
    <div class="ax-swiper-pagination" v-if="itemsArr.length > 1 && indicatorDots"
         :class="vertical === true ? 'ax-swiper-pagination-vertical' : ''">
            <span class="ax-swiper-pagination-item"
                  v-for="(t, i) in itemNums" :key="t"
                  :style="{backgroundColor: paginationIndex === i? indicatorActiveColor : indicatorColor}"
            ></span>
    </div>
  </div>
</template>

<script type="text/babel">

export default {
  name: 'ax-swiper',
  data () {
    return {
      //  轮播图第一张
      firstItem: '',
      //  轮播图最后一张
      lastItem: '',
      //  目前播放的轮播图序号
      currentIndex: 0,
      //  轮播图数量
      itemNums: 0,
      //  存储轮播图数组
      itemsArr: [],
      //  自动播放定时器
      autoPlayTimer: null,
      //  指示点的默认index
      paginationIndex: 0,
      //  滑动设置
      touches: {
        moveTag: 0,
        moveOffset: 0,
        touchStartTime: 0,
        isTouchEvent: false,
        allowClick: false,
        isDraging: false
      },
      //  前边距
      preMargin: parseInt(this.previousMargin),
      //  后边距
      nxtMargin: parseInt(this.nextMargin)
    }
  },
  props: {
    // 是否显示面板指示点
    indicatorDots: {
      type: Boolean,
      default: false
    },
    //  指示点颜色
    indicatorColor: {
      type: String,
      default: '#D8D8D8'
    },
    //  当前选中指示点的颜色
    indicatorActiveColor: {
      type: String,
      default: '#000'
    },
    // 是否自动切换
    autoplay: {
      type: Boolean,
      default: false
    },
    //  当前所在滑块的Index
    current: {
      type: Number,
      default: 0
    },
    //  当前所在滑块的 item-id ，不能与 current 被同时指定(暂未实现）
    currentItemId: {
      type: String,
      default: ''
    },
    // 自动切换时间间隔
    interval: {
      type: Number,
      default: 5000
    },
    // 动画时长
    duration: {
      type: Number,
      default: 500
    },
    //  是否采用链接滑动
    circular: {
      type: Boolean,
      default: false
    },
    // 轮播方向是否为纵向
    vertical: {
      type: Boolean,
      default: false
    },
    //  前边距
    previousMargin: {
      type: String,
      default: '0px'
    },
    //  后边距
    nextMargin: {
      type: String,
      default: '0px'
    },
    //  默认一次性显示的滑块数量（暂未实现）
    displayMultipleItems: {
      type: Number,
      default: 1
    },
    //  是否跳过未显示的滑块布局（不是很明白什么意思，暂不实现）
    skipHiddenItemLayout: {
      type: Boolean,
      default: true
    }
  },
  watch: {
    //  监听默认显示图片的Index
    current (val) {
      val = ~~val
      if (val > this.itemNums) {
        val = this.itemNums
      }
      this.currentIndex = this.circular ? val + 1 : val
      this.showItem(this.currentIndex)
    },
    //  监听图片index的变化(当衔接滚动时，currentindex不变，否则currentindex正常变化)
    currentIndex (val) {
      const itemNums = this.itemNums
      // const tm = (val - 1) % itemNums
      if (this.circular) {
        this.paginationIndex = 0
      } else {
        this.paginationIndex = val < 0 ? itemNums - 1 : val
      }
    },
    //  监听轮播图指示点的变化
    paginationIndex (val) {
      this.$emit('bindchange', {
        detail: {
          current: val,
          currentItemId: '',
          source: 'touch'
        },
        type: 'change'
      })
    },
    previousMargin (val) {
      this.preMargin = parseInt(val)
      this.showItem(1)
    },
    nextMargin (val) {
      this.nxtMargin = parseInt(val)
      this.showItem(1)
    }
  },
  methods: {
    //  轮播图初始化
    init () {
      this.destroy()
      this.isVertical = this.vertical === true
      this.itemsArr = this.$children.filter(item => item.$options.name === 'ax-swiper-item')
      this.itemNums = this.itemsArr.length
      this.currentIndex = 0
      if (this.current > 0) {
        this.currentIndex = ~~this.current
      }
      if (this.circular) {
        this.currentIndex = Math.floor((this.itemsArr.length - 1) / 2)
        if (this.current < Math.floor((this.itemsArr.length - 1) / 2)) {
          let moveItems = this.itemsArr.splice(this.current - Math.floor((this.itemsArr.length - 1) / 2), Math.floor((this.itemsArr.length - 1) / 2) - this.current)
          this.itemsArr = moveItems.concat(this.itemsArr)
        } else {
          let moveItems = this.itemsArr.splice(0, this.current - Math.floor((this.itemsArr.length - 1) / 2))
          this.itemsArr = this.itemsArr.concat(moveItems)
        }
      }
      this.showItem(this.currentIndex)
      this.bindEvents()
      this.autoPlay()
    },
    //  显示默认的轮播图
    showItem (showIndex) {
      if (this.isVertical) {
        this.$refs.swiper.style.height = '100%'
        const itemHeight = this.$refs.wrapper.offsetHeight - this.preMargin - this.nxtMargin
        this.itemsArr.forEach((item, index) => {
          item.$el.style.height = itemHeight + 'px'
          this.setTranslate(item.$el, 0, itemHeight * (index - showIndex) + this.preMargin)
        })
      } else {
        const itemWidth = this.$refs.wrapper.offsetWidth - this.preMargin - this.nxtMargin
        this.itemsArr.forEach((item, index) => {
          item.$el.style.width = itemWidth + 'px'
          this.setTranslate(item.$el, 0, itemWidth * (index - showIndex) + this.preMargin)
        })
      }
    },
    //  滑动开始的操作（记录起始的位置和时间）
    touchStartHandler (event) {
      const touches = this.touches
      touches.allowClick = true
      touches.isTouchEvent = event.type === 'touchstart'
      if (!touches.isTouchEvent && 'which' in event && event.which === 3) return
      if (touches.moveTag === 0) {
        touches.moveTag = 1
        touches.startX = event.touches ? event.touches[0].clientX : event.clientX
        touches.startY = event.touches ? event.touches[0].clientY : event.clientY
        touches.touchStartTime = Date.now()
      }
    },
    //  轮播图滑动过程中的处理
    touchMoveHandler (event) {
      if (!this.supportTouch || this.isVertical) {
        event.preventDefault()
      }
      const touches = this.touches
      const wrapperSize = this.isVertical ? this.$el.clientHeight : this.$refs.wrapper.offsetWidth
      touches.allowClick = false
      if (touches.isTouchEvent && event.type === 'mousemove') return
      const currentY = event.touches ? event.touches[0].clientY : event.clientY
      const currentX = event.touches ? event.touches[0].clientX : event.clientX
      const touchAngle = Math.atan2(Math.abs(currentY - touches.startY), Math.abs(currentX - touches.startX)) * 180 / Math.PI
      const itemWidth = wrapperSize - this.preMargin - this.nxtMargin
      if ((!this.isVertical ? touchAngle > 45 : (90 - touchAngle > 30)) && this.supportTouch) {
        touches.moveTag = 3
        this.stopAutoplay()
        this.setTranslate(0, -(this.currentIndex * (wrapperSize - this.preMargin - this.nxtMargin) - this.preMargin))
        return
      }
      touches.isDraging = true
      const deltaSlide = touches.moveOffset = this.isVertical ? (currentY - touches.startY) : (currentX - touches.startX)
      if (deltaSlide !== 0 && touches.moveTag !== 0) {
        if (touches.moveTag === 1) {
          this.stopAutoplay()
          touches.moveTag = 2
        }
        if (touches.moveTag === 2) {
          if (Math.abs(deltaSlide) * 2 < itemWidth || !this.circular) {
            if (this.isVertical) {
              if (deltaSlide < 0) {
                this.itemsArr[0].$el.style.top = '0px'
              } else {
                this.itemsArr[this.itemNums - 1].$el.style.top = '0px'
              }
            } else {
              if (deltaSlide < 0) {
                this.itemsArr[0].$el.style.left = '0px'
              } else {
                this.itemsArr[this.itemNums - 1].$el.style.left = '0px'
              }
            }
            this.itemsArr.forEach((item, index) => {
              this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin + deltaSlide)
            })
          } else {
            if (deltaSlide < 0) {
              if (this.isVertical) {
                this.itemsArr[0].$el.style.top = itemWidth * this.itemNums + 'px'
              } else {
                this.itemsArr[0].$el.style.left = itemWidth * this.itemNums + 'px'
              }
              this.itemsArr.forEach((item, index) => {
                this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin + deltaSlide)
              })
            } else {
              if (this.isVertical) {
                this.itemsArr[this.itemNums - 1].$el.style.top = -itemWidth * this.itemNums + 'px'
              } else {
                this.itemsArr[this.itemNums - 1].$el.style.left = -itemWidth * this.itemNums + 'px'
              }
              this.itemsArr.forEach((item, index) => {
                this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin + deltaSlide)
              })
            }
          }
        }
      }
    },
    touchEndHandler (event) {
      const touches = this.touches
      const moveOffset = touches.moveOffset
      const wrapperSize = this.isVertical ? this.$el.clientHeight : this.$refs.wrapper.offsetWidth
      const itemWidth = wrapperSize - this.preMargin - this.nxtMargin
      if (touches.moveTag === 1) {
        touches.moveTag = 0
      }
      setTimeout(() => {
        touches.allowClick = true
        touches.isDraging = false
      }, this.duration)
      if (touches.moveTag === 2) {
        touches.moveTag = 0
        const timeDiff = Date.now() - touches.touchStartTime
        const unloopDrag = (!this.circular && ((this.currentIndex === 0 && moveOffset > 0) || (this.currentIndex >= this.itemNums - 1 && moveOffset < 0)))
        if ((timeDiff > 250 && Math.abs(moveOffset) <= itemWidth * 0.5) || (this.itemsArr.length <= 1 || unloopDrag)) {
          this.itemsArr.forEach((item, index) => {
            this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin)
          })
        } else {
          //  当手动向左滑动时
          if (moveOffset < 0) {
            this.paginationIndex += 1
            if (this.paginationIndex > this.itemNums - 1) {
              this.paginationIndex = 0
            }
            this.itemsArr.forEach((item, index) => {
              this.setTranslate(item.$el, this.duration, itemWidth * (index - this.currentIndex - 1) + this.preMargin)
            })
            //  如果是链接滚动
            if (this.circular) {
              setTimeout(() => {
                //  先清除第一张图片的动画效果，否则在实现图片瞬间挪动位置时，会出现动画
                this.itemsArr[0].$el.style.transitionDuration = '0ms'
                if (this.isVertical) {
                  //  如果是纵向，则更改top值，把第一张的位置挪到最后一张的位置去
                  this.itemsArr[0].$el.style.top = itemWidth * this.itemNums + 'px'
                } else {
                  //  否则就更改left值
                  this.itemsArr[0].$el.style.left = itemWidth * this.itemNums + 'px'
                }
                // 把轮播图的第一张追加一份到轮播图的最后面
                this.itemsArr.push(this.itemsArr.shift())
                //  轮播图的所有图片都整体向左移动距离（图片+前后margin距离）
                setTimeout(() => {
                  if (this.isVertical) {
                    this.itemsArr[this.itemsArr.length - 1].$el.style.top = '0px'
                  } else {
                    this.itemsArr[this.itemsArr.length - 1].$el.style.left = '0px'
                  }
                  this.itemsArr.forEach((item, index) => {
                    this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin)
                  })
                  //  传递动画完成事件
                  this.$emit('bindanimationfinish', {
                    detail: {
                      current: this.paginationIndex,
                      currentItemId: '',
                      source: 'touch'
                    },
                    type: 'animationfinish'
                  })
                }, this.duration * 0.5)
              }, this.duration * 0.4)
            } else {
              //  如果不是衔接滚动，则正常加currentindex并传递事件
              this.currentIndex++
              this.$emit('bindanimationfinish', {
                detail: {
                  current: this.paginationIndex,
                  currentItemId: '',
                  source: 'touch'
                },
                type: 'animationfinish'
              })
            }
          } else {
            //  当手动向右滑动轮播图时
            this.paginationIndex -= 1
            if (this.paginationIndex < 0) {
              this.paginationIndex = this.itemNums - 1
            }
            this.itemsArr.forEach((item, index) => {
              this.setTranslate(item.$el, this.duration, itemWidth * (index - this.currentIndex + 1) + this.nxtMargin)
            })
            // 如果是衔接滚动时
            if (this.circular) {
              setTimeout(() => {
                //  先把最后一张的动画清除掉，防止在图片挪动位置的时候，出现动画效果，影响滚动的平滑度
                this.itemsArr[this.itemNums - 1].$el.style.transitionDuration = '0ms'
                //  如果是纵向滚动，则把最后一张图片的top值手动设置一下，让它挪动到第一张的位置
                if (this.isVertical) {
                  this.itemsArr[this.itemNums - 1].$el.style.top = -itemWidth * this.itemNums + 'px'
                //  否则就设置left值
                } else {
                  this.itemsArr[this.itemNums - 1].$el.style.left = -itemWidth * this.itemNums + 'px'
                }
                //  把图片的最后一张追加到数组的第一张
                this.itemsArr.unshift(this.itemsArr.pop())
                setTimeout(() => {
                  if (this.isVertical) {
                    this.itemsArr[0].$el.style.top = '0px'
                  } else {
                    this.itemsArr[0].$el.style.left = '0px'
                  }
                  this.itemsArr.forEach((item, index) => {
                    this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.nxtMargin)
                  })
                  //  传递动画完成事件
                  this.$emit('bindanimationfinish', {
                    detail: {
                      current: this.paginationIndex,
                      currentItemId: '',
                      source: 'touch'
                    },
                    type: 'animationfinish'
                  })
                }, this.duration * 0.5)
              }, this.duration * 0.4)
            } else {
              //  如果不是衔接滑动，则直接减currentindex
              this.$emit('bindanimationfinish', {
                detail: {
                  current: this.paginationIndex,
                  currentItemId: '',
                  source: 'touch'
                },
                type: 'animationfinish'
              })
              this.currentIndex--
            }
          }
        }
        this.autoPlay()
        return
      }
      if (touches.moveTag === 3) {
        touches.moveTag = 0
        this.autoPlay()
      }
    },
    //  自动滚动事件
    autoPlay () {
      if (this.autoplay === false || this.interval <= 0 || this.itemsArr.length <= 1) return
      this.autoPlayTimer = setInterval(() => {
        const size = this.isVertical ? this.$el.clientHeight : this.$refs.wrapper.offsetWidth
        const itemWidth = size - this.preMargin - this.nxtMargin
        this.paginationIndex += 1
        if (this.paginationIndex > this.itemNums - 1) {
          this.paginationIndex = 0
        }
        this.itemsArr.forEach((item, index) => {
          this.setTranslate(item.$el, this.duration, itemWidth * (index - this.currentIndex - 1) + this.preMargin)
        })
        if (this.circular) {
          setTimeout(() => {
            this.itemsArr[0].$el.style.transitionDuration = '0ms'
            if (this.isVertical) {
              this.itemsArr[0].$el.style.top = itemWidth * this.itemNums + 'px'
            } else {
              this.itemsArr[0].$el.style.left = itemWidth * this.itemNums + 'px'
            }
            this.itemsArr.push(this.itemsArr.shift())
            setTimeout(() => {
              if (this.isVertical) {
                this.itemsArr[this.itemsArr.length - 1].$el.style.top = '0px'
              } else {
                this.itemsArr[this.itemsArr.length - 1].$el.style.left = '0px'
              }
              this.itemsArr.forEach((item, index) => {
                this.setTranslate(item.$el, 0, itemWidth * (index - this.currentIndex) + this.preMargin)
              })
              this.$emit('bindanimationfinish', {
                detail: {
                  current: this.paginationIndex,
                  currentItemId: '',
                  source: 'touch'
                },
                type: 'animationfinish'
              })
            }, this.duration * 0.5)
          }, this.duration * 0.4)
        } else {
          this.currentIndex++
          if (this.currentIndex >= this.itemNums - 1) {
            this.currentIndex = -1
          }
          this.$emit('bindanimationfinish', {
            detail: {
              current: this.paginationIndex,
              currentItemId: '',
              source: 'touch'
            },
            type: 'animationfinish'
          })
        }
      }, this.interval)
    },
    sendIndex () {
      if (!this.circular) {
        this.callback && this.callback(this.currentIndex)
      } else {
        let _index = this.currentIndex % this.itemNums
        this.callback && this.callback(_index === 0 ? this.itemNums - 1 : _index - 1)
      }
    },
    stopAutoplay () {
      clearInterval(this.autoPlayTimer)
    },
    stopDrag (event) {
      this.touches.isDraging && event.preventDefault()
    },
    bindEvents () {
      if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
        this.$el.addEventListener('touchstart', this.touchStartHandler)
        this.$el.addEventListener('touchmove', this.touchMoveHandler)
        this.$el.addEventListener('touchend', this.touchEndHandler)
      } else {
        this.$el.addEventListener('mousedown', this.touchStartHandler)
        this.$el.addEventListener('mousemove', this.touchMoveHandler)
        this.$el.addEventListener('mouseup', this.touchEndHandler)
      }
      this.$el.addEventListener('click', (e) => {
        if (!this.touches.allowClick) {
          e.preventDefault()
        }
      }) 
      if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
        document.body.addEventListener('touchmove', this.stopDrag, { passive: false })
      } else {
        document.body.addEventListener('mousemove', this.stopDrag, { passive: false })
      }
    },
    unbindEvents () {
      if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
        this.$el.removeEventListener('touchstart', this.touchStartHandler)
        this.$el.removeEventListener('touchmove', this.touchMoveHandler)
        this.$el.removeEventListener('touchend', this.touchEndHandler)
      } else {
        this.$el.removeEventListener('mousedown', this.touchStartHandler)
        this.$el.removeEventListener('mousemove', this.touchMoveHandler)
        this.$el.removeEventListener('mouseup', this.touchEndHandler)
      }
      if (/Android|webOS|iPhone|iPod|BlackBerry/i.test(navigator.userAgent)) {
        document.body.removeEventListener('touchmove', this.stopDrag, { passive: false })
      } else {
        document.body.removeEventListener('mousemove', this.stopDrag, { passive: false })
      }
    },
    setTranslate (element, speed, translate) {
      element.style.transitionDuration = speed + 'ms'
      if (this.isVertical) {
        element.style.transform = 'translate3d(0, ' + translate + 'px, 0)'
      } else {
        element.style.transform = 'translate3d(' + translate + 'px, 0, 0)'
      }
    },
    destroy () {
      this.unbindEvents()
      this.stopAutoplay()
    }
  },
  beforeDestroy () {
    this.destroy()
  }
}
</script>
<style scoped>
  @keyframes yd-kf-opacity-in {
    0% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  }

  .ax-swiper {
    width: 100%;
    overflow: hidden;
    position: relative;
    height: 200px;
  }

  .ax-swiper-wrapper {
    display: flex;
    width: 100%;
    height: 100%;
    /*transform: translate3d(0px, 0px, 0px);*/
    position: relative;
    z-index: 1;
    align-items: center;
    /*transition-property: transform;*/
  }

  .ax-swiper-wrapper-vertical {
    flex-direction: column;
  }

  .ax-swiper-item {
    position: absolute;
    width: 100%;
    height: 100%;
    flex-shrink: 0;
    will-change: transform, left;
  }

  .ax-swiper-item a {
    display: block;
  }

  .ax-swiper-item img {
    width: 100%;
    display: block;
  }

  .ax-swiper-pagination {
    position: absolute;
    width: 100%;
    z-index: 2;
    left: 0;
    bottom: .1rem;
    pointer-events: none;
    display: flex;
    align-items: flex-end;
    justify-content: center;
    opacity: 0;
    animation: yd-kf-opacity-in .3s linear .4s forwards;
  }

  .ax-swiper-pagination-vertical {
    width: 0;
    height: 100%;
    flex-direction: column;
    bottom: 0;
    left: auto;
    right: .8rem;
    justify-content: center;
  }

  .ax-swiper-pagination-item {
    margin: 0.3rem;
    margin-bottom: 0.8rem;
    width: 9px;
    height: 9px;
    display: inline-block;
    border-radius: 100%;
  }
</style>
