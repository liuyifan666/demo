<script setup>
import { computed, ref } from 'vue'
import { ElMessage } from 'element-plus'

const asset = path => `${import.meta.env.BASE_URL}${path}`

const gallery = [
  { src: asset('assets/figma-16.png'), label: '主图' },
  { src: asset('assets/figma-01.png'), label: '包装' },
  { src: asset('assets/figma-04.png'), label: '包装侧面' },
  { src: asset('assets/figma-06.png'), label: '食材' },
]
const isFlag = ref(true)
const activeImageIndex = ref(0)
const activeTab = ref('recommend')
const galleryCollapsed = ref(false)
const mainImage = computed(() => gallery[activeImageIndex.value].src)

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

function selectImage(index) {
  isFlag.value = !index
  activeImageIndex.value = index
}

function toggleGallery() {
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
      <section class="hero-media">
        <img class="hero-image" :src="mainImage" :alt="gallery[activeImageIndex].label" />
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
        <div class="play-button" v-show="isFlag">
          <img class="main-play-icon" :src="asset('assets/icons/play.svg')" alt="" aria-hidden="true" />
        </div>
        <div class="gallery-controls" :class="{ 'is-collapsed': galleryCollapsed }">
          <div class="image-index">{{ activeImageIndex + 1 }} / {{ gallery.length }}</div>
          <div class="thumb-strip">
            <button
              v-for="(image, index) in gallery.slice(0, 3)"
              :key="image.src"
              class="thumb"
              :class="{ active: activeImageIndex === index }"
              type="button"
              @click="selectImage(index)"
            >
              <img :src="image.src" :alt="image.label" />
              <span v-if="index === 0" class="thumb-play" aria-hidden="true">
                <img class="thumb-play-icon" :src="asset('assets/icons/play2.svg')" alt="" />
              </span>
            </button>
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
