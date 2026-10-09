<script setup>
import { nextTick, onMounted, onUnmounted, ref } from 'vue'
import { ElMessage } from 'element-plus'

const asset = path => `${import.meta.env.BASE_URL}${path}`

const gallery = [
  { src: asset('assets/figma-16.png'), label: '主图' },
  { src: asset('assets/figma-01.png'), label: '包装' },
  { src: asset('assets/figma-04.png'), label: '包装侧面' },
  { src: asset('assets/figma-11.png'), label: '食材' },
]
const isFlag = ref(true)
const activeImageIndex = ref(0)
const activeTab = ref('recommend')
const galleryCollapsed = ref(true)
const headerPinned = ref(false)
const heroMediaRef = ref(null)
const heroDragOffset = ref(0)
const heroDragging = ref(false)
const thumbStripRef = ref(null)
const thumbTrackRef = ref(null)
const thumbDragOffset = ref(0)
const thumbDragging = ref(false)
let thumbGesture = null
let suppressThumbClick = false
let heroGesture = null

function updateHeaderPinned() {
  const heroMedia = heroMediaRef.value
  headerPinned.value = Boolean(heroMedia && heroMedia.getBoundingClientRect().bottom <= 0)
}

function dragDirection(deltaX, deltaY) {
  const x = Math.abs(deltaX)
  const y = Math.abs(deltaY)
  if (Math.max(x, y) < 8) {
    return null
  }
  if (x > y * 1.2) {
    return 'horizontal'
  }
  if (y > x * 1.2) {
    return 'vertical'
  }
  return null
}

function createHeroGesture(element, pointerId, clientX, clientY, input) {
  return {
    input,
    pointerId,
    element,
    startX: clientX,
    startY: clientY,
    width: element.clientWidth,
    index: activeImageIndex.value,
    dragging: false,
  }
}

function updateHeroGesture(gesture, clientX, clientY, event) {
  const deltaX = clientX - gesture.startX
  const deltaY = clientY - gesture.startY
  if (!gesture.dragging) {
    const direction = dragDirection(deltaX, deltaY)
    if (!direction) {
      return
    }
    if (direction === 'vertical') {
      finishHeroDrag()
      return
    }
    gesture.dragging = true
    heroDragging.value = true
  }

  if (event.cancelable) {
    event.preventDefault()
  }

  const atEdge = (gesture.index === 0 && deltaX > 0) ||
    (gesture.index === gallery.length - 1 && deltaX < 0)
  heroDragOffset.value = atEdge
    ? deltaX * 0.25
    : Math.max(-gesture.width, Math.min(gesture.width, deltaX))
}

function startHeroDrag(event) {
  if (event.pointerType === 'touch' || !event.isPrimary || event.button !== 0 || heroGesture) {
    return
  }

  heroGesture = createHeroGesture(
    event.currentTarget,
    event.pointerId,
    event.clientX,
    event.clientY,
    'pointer',
  )
  event.currentTarget.setPointerCapture(event.pointerId)
}

function moveHeroDrag(event) {
  const gesture = heroGesture
  if (!gesture || gesture.input !== 'pointer' || gesture.pointerId !== event.pointerId) {
    return
  }

  updateHeroGesture(gesture, event.clientX, event.clientY, event)
}

function startHeroTouch(event) {
  if (event.touches.length !== 1) {
    finishHeroDrag()
    return
  }
  if (heroGesture) {
    return
  }

  const touch = event.changedTouches[0]
  if (!touch) {
    return
  }

  heroGesture = createHeroGesture(
    event.currentTarget,
    touch.identifier,
    touch.clientX,
    touch.clientY,
    'touch',
  )
}

function moveHeroTouch(event) {
  const gesture = heroGesture
  if (!gesture || gesture.input !== 'touch') {
    return
  }
  if (event.touches.length !== 1) {
    finishHeroDrag()
    return
  }

  const touch = Array.from(event.touches).find(item => item.identifier === gesture.pointerId)
  if (!touch) {
    return
  }

  updateHeroGesture(gesture, touch.clientX, touch.clientY, event)
}

function finishHeroDrag(event) {
  const gesture = heroGesture
  if (!gesture || (event && (gesture.input !== 'pointer' ||
    event.pointerType === 'touch' || gesture.pointerId !== event.pointerId))) {
    return
  }
  if (event?.type === 'lostpointercapture' && event.target !== gesture.element) {
    return
  }

  heroGesture = null
  heroDragging.value = false
  heroDragOffset.value = 0
  if (gesture.input === 'pointer' && gesture.element.hasPointerCapture(gesture.pointerId)) {
    gesture.element.releasePointerCapture(gesture.pointerId)
  }

  if (gesture.dragging && event?.type === 'pointerup') {
    const distance = event.clientX - gesture.startX
    if (Math.abs(distance) >= Math.min(80, gesture.width * 0.2)) {
      const index = Math.max(0, Math.min(gallery.length - 1,
        gesture.index + (distance < 0 ? 1 : -1)))
      if (index !== gesture.index) {
        selectImage(index)
      }
    }
  }
}

function finishHeroTouch(event) {
  const gesture = heroGesture
  if (!gesture || gesture.input !== 'touch') {
    return
  }

  const touch = Array.from(event.changedTouches).find(item => item.identifier === gesture.pointerId)
  if (!touch) {
    return
  }

  const distance = touch.clientX - gesture.startX
  const shouldChange = event.type === 'touchend' && gesture.dragging &&
    Math.abs(distance) >= Math.min(80, gesture.width * 0.2)
  finishHeroDrag()

  if (shouldChange) {
    const index = Math.max(0, Math.min(gallery.length - 1,
      gesture.index + (distance < 0 ? 1 : -1)))
    if (index !== gesture.index) {
      selectImage(index)
    }
  }
}

function createThumbGesture(element, pointerId, clientX, clientY, input) {
  const strip = thumbStripRef.value
  const track = thumbTrackRef.value
  suppressThumbClick = false
  if (galleryCollapsed.value || !strip || !track) {
    return
  }

  const style = window.getComputedStyle(strip)
  const padding = parseFloat(style.paddingLeft) + parseFloat(style.paddingRight)
  thumbGesture = {
    input,
    pointerId,
    element,
    startX: clientX,
    startY: clientY,
    scrollLeft: strip.scrollLeft,
    maxScrollLeft: Math.max(0, track.offsetWidth + padding - strip.clientWidth),
    dragging: false,
  }
}

function startThumbDrag(event) {
  if (event.pointerType === 'touch' || !event.isPrimary || event.button !== 0 || thumbGesture) {
    return
  }
  createThumbGesture(event.currentTarget, event.pointerId, event.clientX, event.clientY, 'pointer')
}

function moveThumbDrag(event) {
  const gesture = thumbGesture
  if (!gesture || gesture.input !== 'pointer' || gesture.pointerId !== event.pointerId) {
    return
  }

  updateThumbGesture(gesture, event.clientX, event.clientY, event)
}

function updateThumbGesture(gesture, clientX, clientY, event) {
  const deltaX = clientX - gesture.startX
  const deltaY = clientY - gesture.startY
  if (!gesture.dragging) {
    const direction = dragDirection(deltaX, deltaY)
    if (!direction) {
      return
    }
    if (direction === 'vertical') {
      finishThumbDrag()
      return
    }
    gesture.dragging = true
    thumbDragging.value = true
    if (gesture.input === 'pointer') {
      gesture.element.setPointerCapture(event.pointerId)
    }
  }

  if (event.cancelable) {
    event.preventDefault()
  }

  // Keep the scroll range stable while the inner track is stretched.
  const proposed = gesture.scrollLeft - deltaX
  const clamped = Math.max(0, Math.min(gesture.maxScrollLeft, proposed))
  gesture.element.scrollLeft = clamped
  const overscroll = clamped - proposed
  thumbDragOffset.value = Math.sign(overscroll) * 48 * (1 - Math.exp(-Math.abs(overscroll) / 100))
}

function finishThumbDrag(event) {
  const gesture = thumbGesture
  if (!gesture || (event && (gesture.input !== 'pointer' ||
    event.pointerType === 'touch' || gesture.pointerId !== event.pointerId))) {
    return
  }

  // A child can lose its implicit capture when the strip takes over dragging.
  if (event?.type === 'lostpointercapture' && event.target !== gesture.element) {
    return
  }

  thumbGesture = null
  suppressThumbClick = gesture.dragging
  thumbDragging.value = false
  thumbDragOffset.value = 0
  if (gesture.input === 'pointer' && gesture.element.hasPointerCapture(gesture.pointerId)) {
    gesture.element.releasePointerCapture(gesture.pointerId)
  }
}

function startThumbTouch(event) {
  if (event.touches.length !== 1) {
    finishThumbDrag()
    return
  }
  if (thumbGesture) {
    return
  }
  const touch = event.changedTouches[0]
  if (touch) {
    createThumbGesture(event.currentTarget, touch.identifier, touch.clientX, touch.clientY, 'touch')
  }
}

function moveThumbTouch(event) {
  const gesture = thumbGesture
  if (!gesture || gesture.input !== 'touch') {
    return
  }
  if (event.touches.length !== 1) {
    finishThumbDrag()
    return
  }
  const touch = Array.from(event.touches).find(item => item.identifier === gesture.pointerId)
  if (touch) {
    updateThumbGesture(gesture, touch.clientX, touch.clientY, event)
  }
}

function finishThumbTouch(event) {
  if (thumbGesture?.input === 'touch' &&
    Array.from(event.changedTouches).some(item => item.identifier === thumbGesture.pointerId)) {
    finishThumbDrag()
  }
}

function guardThumbClick(event) {
  if (suppressThumbClick && event.detail !== 0) {
    event.preventDefault()
    event.stopPropagation()
    suppressThumbClick = false
  }
}

onMounted(() => {
  updateHeaderPinned()
  window.addEventListener('scroll', updateHeaderPinned, { passive: true })
  window.addEventListener('resize', updateHeaderPinned, { passive: true })
})

onUnmounted(() => {
  window.removeEventListener('scroll', updateHeaderPinned)
  window.removeEventListener('resize', updateHeaderPinned)
  finishThumbDrag()
  finishHeroDrag()
})

const prices = ref([
  {
    name: '一件 (10包)',
    unit: '¥6.54/斤',
    price: '¥133.97',
    count: 3,
    gifts: ['每满5包减4.1 (¥40/包)'],
  },
  {
    name: '一件 (10包)',
    unit: '¥6.35/斤',
    price: '¥146.72',
    original: '¥159.67',
    count: 1,
    gifts: ['每满5包减4.1 (¥40/包)', '每满10包减6.1 (¥38/包)'],
  },
])

const recommendations = ref([
  {
    image: asset('assets/figma-09.png'),
    title: '[新西兰黑牌] 牛肚片片',
    price: '¥2533.97/包',
    tag: '易解冻',
    metaTags: ['特价', '限时折扣', '领¥30.88券运'],
    showPurchased: true,
    cartQuantity: 1,
  },
  {
    image: asset('assets/figma-06.png'),
    title: '[长江桂柳] 大白条鸭',
    price: '¥71.18/斤',
    tag: '易解冻',
    metaTags: ['满赠'],
    showPurchased: false,
    cartQuantity: 1,
  },
  {
    image: asset('assets/figma-01.png'),
    title: '[九帝] 西装鸡整箱',
    price: '¥14.40/斤',
    tag: '限时折扣',
    metaTags: ['限时折扣'],
    showPurchased: false,
    cartQuantity: 1,
  },
])

const faqs = [
  {
    question: '购买运费如何收取？',
    paragraphs: [
      '自营车队配送范围内单笔订单金额（不含运费）满199元免配送费，',
      '单笔订单不满199元，每单收取9元配送费。',
    ],
  },
  {
    question: '使用什么物流发货？',
    paragraphs: [
      '晓餐同城配送车队可覆盖至广州、佛山、中山、东莞、深圳',
      '其他周边城市可以走物流，详情在线客服',
    ],
  },
  {
    question: '如何申请退货？',
    paragraphs: [
      '请联系客服申请无质量问题退货，退款原路返还，',
      '一般退货次日到账，不同的银行处理时间不同，',
      '周日财务休息不处理退款',
    ],
  },
  {
    question: '可以指定配送时间吗？',
    paragraphs: ['为保证配送效率，按照优先线路沿途配送，无法指定具体到达时间'],
  },
  {
    question: '退换货规则',
    sections: [
      {
        label: '城配订单',
        tone: 'orange',
        paragraphs: [
          '1.质量问题请于收货72小时内及时联系客服申请售后',
          '2.非质量问题在收货48小时内联系客服申请售后，请确保产品未拆包且包装完好；非质量问题拒收或联系不上导致退货，运费只做部分退款',
          '3.易解冻损耗产品（鸡柳，水饺，汤圆，肥牛羊卷，骨肉相连，奥尔良腿排，水产海鲜等）非质量问题不支持无理由退换',
          '4.收货时请与配送员当场清点商品数量及开箱验货，不满意当场退货，过后告知数量不足平台无法处理',
        ],
      },
      {
        label: '物流订单',
        tone: 'blue',
        paragraphs: [
          '1.冷冻食品属于特殊商品，物流揽收后不支持修改收货地址和非质量问题退换货',
          '2.下单后请保持电话畅通，偶有网点将快递放在自提点，需自提，不得以此为由拒收',
          '3.因买家原因（含电话地址错误、电话无人接听导致的延误等）商品退回，拒收或存放自提点过久导致的解冻变质，一切损失由客户承担，将拒绝退款和售后',
          '4.发货采用泡沫箱加冰袋发货，物流途中包装可能被冰袋浸湿，属于正常情况，不影响商品品质，不予售后',
        ],
      },
    ],
  },
]

const cartCount = ref(1)

async function selectImage(index) {
  finishHeroDrag()
  isFlag.value = !index
  activeImageIndex.value = index
  await nextTick()

  const strip = thumbStripRef.value
  const thumbnail = thumbTrackRef.value?.children[index]
  if (!strip || !thumbnail || galleryCollapsed.value) {
    return
  }

  const viewport = strip.getBoundingClientRect()
  const image = thumbnail.getBoundingClientRect()
  const padding = window.getComputedStyle(strip)
  const left = viewport.left + parseFloat(padding.paddingLeft)
  const right = viewport.right - parseFloat(padding.paddingRight)
  if (image.left < left || image.right > right) {
    strip.scrollTo({
      left: strip.scrollLeft + (image.left < left ? image.left - left : image.right - right),
      behavior: 'smooth',
    })
  }
}

function toggleGallery() {
  finishThumbDrag()
  galleryCollapsed.value = !galleryCollapsed.value
}

function updateCount(index, delta) {
  prices.value[index].count = Math.max(1, prices.value[index].count + delta)
}

function addToCart(item) {
  if (item) {
    item.cartQuantity += 1
  }
  cartCount.value += 1
  ElMessage({
    message: '已加入购物车',
    type: 'success',
    plain: true,
  })
}

function scrollToTop() {
  window.scrollTo({
    top: 0,
    behavior: 'smooth',
  })
}

</script>

<template>
  <div class="app-stage">
    <main class="phone-page">
      <header class="page-header" :class="{ 'is-pinned': headerPinned }">
        <img class="status-bar-image" :src="asset('assets/figma-17.png')" alt="" aria-hidden="true" />
        <div class="media-tools">
          <div class="media-nav-pill">
            <el-button class="media-nav-button" aria-label="返回" title="返回">
              <img class="media-arrow-icon" :src="asset('assets/icons/left.svg')" alt="" aria-hidden="true" />
            </el-button>
            <span class="media-nav-divider" aria-hidden="true"></span>
            <el-button class="media-nav-button" aria-label="分享" title="分享">
              <img class="media-share-icon" :src="asset('assets/icons/share.svg')" alt="" aria-hidden="true" />
            </el-button>
          </div>
          <div class="media-spacer"></div>
          <div class="media-options-pill">
            <el-button circle class="media-more-button" aria-label="功能" title="更多">
              <img class="media-function-icon" :src="asset('assets/icons/function.svg')" alt="" aria-hidden="true" />
            </el-button>
            <span class="media-options-divider" aria-hidden="true"></span>
            <button class="lens-button" type="button" aria-label="关闭" title="查看状态">
              <img class="lens-icon" :src="asset('assets/icons/close.svg')" alt="" aria-hidden="true" />
            </button>
          </div>
        </div>
      </header>
      <section ref="heroMediaRef" class="hero-media">
        <div
          class="hero-image-viewport"
          :class="{ 'is-dragging': heroDragging }"
          @pointerdown="startHeroDrag"
          @pointermove="moveHeroDrag"
          @pointerup="finishHeroDrag"
          @pointercancel="finishHeroDrag"
          @lostpointercapture="finishHeroDrag"
          @touchstart="startHeroTouch"
          @touchmove="moveHeroTouch"
          @touchend="finishHeroTouch"
          @touchcancel="finishHeroTouch"
        >
          <div
            class="hero-image-track"
            :class="{ 'is-dragging': heroDragging }"
            :style="{ transform: `translateX(calc(-${activeImageIndex * 100}% + ${heroDragOffset}px))` }"
          >
            <img
              v-for="image in gallery"
              :key="image.src"
              class="hero-image"
              :src="image.src"
              :alt="image.label"
              draggable="false"
            />
          </div>
        </div>
        <div class="play-button" v-show="isFlag">
          <img class="main-play-icon" :src="asset('assets/icons/play.svg')" alt="" aria-hidden="true" />
        </div>
        <div class="gallery-controls" :class="{ 'is-collapsed': galleryCollapsed }">
          <div class="image-index">{{ activeImageIndex + 1 }} / {{ gallery.length }}</div>
          <div
            ref="thumbStripRef"
            class="thumb-strip"
            @pointerdown="startThumbDrag"
            @pointermove="moveThumbDrag"
            @pointerup="finishThumbDrag"
            @pointercancel="finishThumbDrag"
            @lostpointercapture="finishThumbDrag"
            @touchstart="startThumbTouch"
            @touchmove="moveThumbTouch"
            @touchend="finishThumbTouch"
            @touchcancel="finishThumbTouch"
            @click.capture="guardThumbClick"
          >
            <div
              ref="thumbTrackRef"
              class="thumb-track"
              :class="{ 'is-dragging': thumbDragging }"
              :style="{ transform: `translateX(${thumbDragOffset}px)` }"
            >
              <button
                v-for="(image, index) in gallery"
                :key="image.src"
                class="thumb"
                :class="{ active: activeImageIndex === index }"
                :tabindex="galleryCollapsed && activeImageIndex !== index ? -1 : 0"
                :aria-hidden="galleryCollapsed && activeImageIndex !== index"
                type="button"
                @click="selectImage(index)"
              >
                <img :src="image.src" :alt="image.label" draggable="false" />
                <span v-if="index === 0" class="thumb-play" aria-hidden="true">
                  <img class="thumb-play-icon" :src="asset('assets/icons/play2.svg')" alt="" draggable="false" />
                </span>
              </button>
            </div>
          </div>
          <el-button
            circle
            class="thumb-next"
            :aria-label="galleryCollapsed ? '展开缩略图' : '收起缩略图'"
            :title="galleryCollapsed ? '展开缩略图' : '收起缩略图'"
            @click="toggleGallery"
          >
            <img
              class="gallery-arrow-icon"
              :class="{ rotated: galleryCollapsed }"
              :src="asset('assets/icons/left.svg')"
              alt=""
              aria-hidden="true"
            />
          </el-button>
        </div>
      </section>

      <section class="content-card product-card">
        <div class="product-heading">
          <div>
            <h1>[巴西93厂] 牛腩</h1>
            <div class="tag-row">
              <el-tag effect="plain" class="tag tag-orange">新品</el-tag>
              <el-tag effect="plain" class="tag tag-green">清真</el-tag>
              <el-tag effect="plain" class="tag tag-blue">易解冻</el-tag>
              <el-tag effect="plain" class="tag tag-red">特价</el-tag>
            </div>
          </div>
          <!-- <span class="product-code">1625068680</span> -->
        </div>
        <p class="subline">猪肉馅包子馅子，点心粉粉半成品</p>
        <div class="stats-grid">
          <div class="stat-cell">
            <strong>质检报告</strong>
            <span>查看 <img class="content-arrow-icon" :src="asset('assets/icons/right.svg')" alt="" aria-hidden="true" /></span>
          </div>
          <div class="stat-cell">
            <strong>中国</strong>
            <span>产地</span>
          </div>
          <div class="stat-cell">
            <strong>1.6kg</strong>
            <span>规格</span>
          </div>
          <div class="stat-cell">
            <strong>365天</strong>
            <span>保质期</span>
          </div>
          <div class="stat-cell">
            <strong>冷冻</strong>
            <span>贮存</span>
          </div>
        </div>
      </section>

      <section class="content-card logistics-card">
        <div class="info-row">
          <span class="row-label">送至</span>
          <div class="row-main">
            <strong>碗碗香麻辣烫</strong>
            <p>广州市番禺区汉溪长隆G区25号奈雪的茶一...</p>
          </div>
        </div>
        <div class="info-row arrow-row">
          <span class="row-label">售后 <em>*</em></span>
          <div class="row-main">
            <strong>非质量问题不支持无理由退换货</strong>
          </div>
          <img class="content-arrow-icon row-arrow-icon" :src="asset('assets/icons/right.svg')" alt="" aria-hidden="true" />
        </div>
        <div class="info-row activity-row">
          <span class="row-label">活动</span>
          <div class="activity-list">
            <div class="coupon-row">
              <span class="coupon">满28.88减2.88</span>
              <span class="coupon">满88.88减8.88</span>
              <span class="coupon">满288.88减</span>
              <img class="content-arrow-icon activity-arrow-icon" :src="asset('assets/icons/right.svg')" alt="" aria-hidden="true" />
            </div>
            <div class="activity-text">
              <span class="coupon coupon-strong">满赠</span>
              <span>每买满1件送[安井] 黄金蛋馄饨1件、[鹰宏亿] 年糕小串1件</span>
            </div>
          </div>
        </div>
      </section>

      <section class="content-card price-card">
        <article v-for="(item, index) in prices" :key="`${item.price}-${index}`" class="price-item">
          <div class="price-head">
            <div class="price-name">
              <strong>{{ item.name }}</strong>
              <span class="unit">{{ item.unit }}</span>
            </div>
            <div class="quantity-control">
              <button
                class="quantity-image-button"
                :disabled="item.count <= 1"
                aria-label="减少数量"
                @click="updateCount(index, -1)"
              >
                <img :src="asset('assets/reduce.png')" alt="" aria-hidden="true" />
              </button>
              <span>{{ item.count }}</span>
              <button
                class="quantity-image-button"
                aria-label="增加数量"
                @click="updateCount(index, 1)"
              >
                <img :src="asset('assets/add.png')" alt="" aria-hidden="true" />
              </button>
            </div>
          </div>
          <div class="price-line">
            <strong>{{ item.price }}</strong>
            <del v-if="item.original">{{ item.original }}</del>
          </div>
          <div v-for="gift in item.gifts" :key="gift" class="gift-line">
            <span>{{ gift }}</span>
            <el-tag effect="dark" type="success">+{{ gift.includes('10包') ? '10' : '5' }}</el-tag>
          </div>
        </article>
      </section>

      <section class="ranking-bar content-card">
        <span class="ranking-mark">榜</span>
        <strong>晓餐推荐卤味鸭货--鸭腿 第<span>3</span>名</strong>
        <img class="content-arrow-icon ranking-arrow-icon" :src="asset('assets/icons/right.svg')" alt="" aria-hidden="true" />
      </section>

      <section class="content-card recommend-card">
        <div class="section-tabs" :class="{ 'is-similar': activeTab === 'similar' }">
          <button
            class="section-tab"
            :class="{ active: activeTab === 'recommend' }"
            type="button"
            @click="activeTab = 'recommend'"
          >
            推荐加购
          </button>
          <button
            class="section-tab"
            :class="{ active: activeTab === 'similar' }"
            type="button"
            @click="activeTab = 'similar'"
          >
            相似商品
          </button>
          <span class="section-tab-indicator" aria-hidden="true"></span>
        </div>
        <div class="recommend-scroller">
          <article v-for="item in recommendations" :key="item.title" class="recommend-item">
            <div class="recommend-image-wrap">
              <img :src="item.image" :alt="item.title" />
              <img
                v-if="item.title.includes('牛肚')"
                class="recommend-brand-mark"
                :src="asset('assets/brand.png')"
                alt="清真"
              />
              <span v-if="item.showPurchased" class="recommend-tag recommend-purchased-tag">买过</span>
              <span class="recommend-tag recommend-condition-tag">{{ item.tag }}</span>
            </div>
            <h3>{{ item.title }}</h3>
            <div class="recommend-meta">
              <span v-for="metaTag in item.metaTags" :key="metaTag" class="mini-label">
                {{ metaTag }}
              </span>
            </div>
            <div class="recommend-price">
              <strong>{{ item.price }}</strong>
              <el-badge
                :value="item.cartQuantity"
                :hidden="item.cartQuantity === 0"
                type="danger"
                class="mini-cart-badge"
              >
                <el-button circle class="mini-cart" aria-label="加入购物车" @click="addToCart(item)">
                  <img :src="asset('assets/icons/shopCart2.svg')" alt="" aria-hidden="true" />
                </el-button>
              </el-badge>
            </div>
          </article>
        </div>
      </section>

      <section class="content-card detail-card">
        <h2>商品详情</h2>
        <div class="detail-poster" :style="{ backgroundImage: `url(${asset('assets/bg.png')})` }">
          <!-- <div class="poster-pills">
            <span><b>海产</b><small>当日采购</small></span>
            <span><b>猪肉</b><small>集团专供</small></span>
            <span><b>蔬菜</b><small>新鲜采购</small></span>
            <span><b>面粉</b><small>优质小麦粉</small></span>
            <span><b>制作</b><small>传统手工</small></span>
          </div> -->
        </div>
      </section>

      <section class="content-card faq-card">
        <h2>常见问题</h2>
        <article v-for="item in faqs" :key="item.question" class="faq-item">
          <div class="faq-question">
            <span class="faq-dot">?</span>
            <strong>{{ item.question }}</strong>
          </div>
          <div v-if="item.paragraphs" class="faq-answer">
            <p v-for="paragraph in item.paragraphs" :key="paragraph">{{ paragraph }}</p>
          </div>
          <div v-for="section in item.sections" :key="section.label" class="faq-rule-section">
            <span class="faq-rule-tag" :class="`faq-rule-tag-${section.tone}`">{{ section.label }}</span>
            <div class="faq-answer">
              <p v-for="paragraph in section.paragraphs" :key="paragraph">{{ paragraph }}</p>
            </div>
          </div>
        </article>
      </section>

      <div class="bottom-space"></div>
    </main>

    <button class="back-to-top" type="button" aria-label="回到顶部" @click="scrollToTop">
      <img :src="asset('assets/icons/top.svg')" alt="" aria-hidden="true" />
      <span>顶部</span>
    </button>

    <footer class="bottom-action">
      <div class="bottom-nav">
        <button class="bottom-nav-item" type="button" aria-label="客服">
          <img :src="asset('assets/icons/service.svg')" alt="" aria-hidden="true" />
          <span>客服</span>
        </button>
        <button class="bottom-nav-item" type="button" aria-label="收藏">
          <img :src="asset('assets/icons/collection.svg')" alt="" aria-hidden="true" />
          <span>收藏</span>
        </button>
        <button class="bottom-nav-item" type="button" aria-label="购物车">
          <el-badge :value="cartCount" :hidden="cartCount === 0" type="danger">
            <img :src="asset('assets/icons/shopCart.svg')" alt="" aria-hidden="true" />
          </el-badge>
          <span>购物车</span>
        </button>
      </div>
      <button class="cart-action" type="button" @click="addToCart">加入购物车</button>
    </footer>
  </div>
</template>
