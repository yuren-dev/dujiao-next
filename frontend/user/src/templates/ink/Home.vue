<template>
  <div>
    <!-- ==================== 列表模式 ==================== -->
    <template v-if="isListMode">
      <section class="mx-auto w-full max-w-[1400px] px-[var(--ink-edge)] py-6">
        <div class="grid items-start gap-5 lg:grid-cols-[248px_1fr] lg:gap-7">
          <InkCategorySidebar
            :category-groups="categoryGroups"
            :selected-category="selectedCategory"
            :expanded-parent-ids="expandedParentIds"
            @select="selectCategory"
            @toggle="toggleParentCategory"
          />

          <main class="min-w-0">
            <div class="relative mb-4">
              <Search class="pointer-events-none absolute left-3.5 top-1/2 h-[18px] w-[18px] -translate-y-1/2 text-muted-foreground" />
              <input
                v-model="searchQuery"
                class="h-11 w-full border bg-card pl-10 pr-10 text-[15px] text-foreground transition-colors placeholder:text-muted-foreground hover:border-[color:var(--ink-border-hover)]"
                style="border-color: var(--ink-border)"
                :placeholder="t('products.searchBoxPlaceholder')"
                :aria-label="t('products.searchLabel')"
              />
              <button v-if="searchQuery" type="button" class="absolute right-2.5 top-1/2 grid h-6 w-6 -translate-y-1/2 place-items-center text-muted-foreground transition-colors hover:text-foreground" :aria-label="t('blog.searchClear')" @click="clearSearch"><X class="h-3.5 w-3.5" /></button>
            </div>

            <div v-if="listLoading" class="space-y-2.5">
              <div v-for="i in 8" :key="i" class="h-[76px] border bg-card" style="border-color: var(--ink-border)"></div>
            </div>

            <div v-else-if="listProductGroups.length" class="space-y-6">
              <div v-for="group in listProductGroups" :key="group.categoryId ?? 'uncategorized'">
                <div class="mb-2.5 flex items-center gap-2 px-0.5">
                  <span class="ink-dot-rule h-[1px] w-6 flex-none"></span>
                  <img v-if="group.categoryIcon" :src="getImageUrl(group.categoryIcon)" :alt="group.categoryName" loading="lazy" class="h-5 w-5 flex-none object-cover" />
                  <span class="min-w-0 truncate text-sm font-bold text-foreground">{{ group.categoryName }}</span>
                  <span class="flex-none text-xs text-muted-foreground">({{ group.products.length }})</span>
                </div>
                <div class="space-y-2">
                  <InkProductListItem
                    v-for="product in group.products"
                    :key="product.id"
                    :product="product"
                    @quick-buy="openQuickBuy"
                  />
                </div>
              </div>

              <nav v-if="listTotalPages > 1" class="mt-[30px] flex flex-wrap justify-center gap-2">
                <button class="ink-btn ink-btn-ghost !h-[42px] !min-w-[42px] disabled:cursor-not-allowed disabled:opacity-40" :disabled="listCurrentPage <= 1" @click="listChangePage(listCurrentPage - 1)"><ChevronLeft class="h-[17px] w-[17px]" /></button>
                <button
                  v-for="p in listPageWindow"
                  :key="p"
                  class="ink-btn !h-[42px] !min-w-[42px]"
                  :class="p === listCurrentPage ? 'ink-btn-primary' : 'ink-btn-ghost'"
                  @click="listChangePage(p)"
                >{{ p }}</button>
                <button class="ink-btn ink-btn-ghost !h-[42px] !min-w-[42px] disabled:cursor-not-allowed disabled:opacity-40" :disabled="listCurrentPage >= listTotalPages" @click="listChangePage(listCurrentPage + 1)"><ChevronRight class="h-[17px] w-[17px]" /></button>
              </nav>
            </div>

            <div v-else class="flex flex-col items-center gap-3 border border-dashed py-14 text-center text-muted-foreground" style="border-color: var(--ink-border)">
              <component :is="(searchQuery || selectedCategory) ? SearchX : PackageOpen" class="h-12 w-12 opacity-60" />
              <p>{{ (searchQuery || selectedCategory) ? t('products.emptyFiltered') : t('products.empty') }}</p>
              <button v-if="searchQuery || selectedCategory" type="button" class="ink-btn ink-btn-ghost mt-2 !h-9 !text-sm" @click="resetFilters">{{ t('products.clearFilters') }}</button>
            </div>
          </main>
        </div>
      </section>
    </template>

    <!-- ==================== 卡片模式（默认） ==================== -->
    <template v-else>
      <!-- Hero：站长品牌 + 现有 home.hero 文案，不新增文案 key -->
      <section class="ink-section !pb-0">
        <div class="ink-stagger" :class="{ 'is-shown': true }">
          <p class="ink-stagger-line mx-auto w-fit bg-[color:var(--ink-amber-soft)] px-3 py-1 text-[13px] font-semibold text-[color:var(--ink-amber-deep)]">{{ t('home.hero.badge') }}</p>
          <h1 class="ink-stagger-line ink-stagger-line--2 mx-auto mt-5 max-w-[980px] text-center font-semibold" style="font-size: clamp(2.1rem, 4.6vw, 3.5rem); line-height: 1.18; letter-spacing: -0.01em;">{{ brandName || t('home.hero.title') }}</h1>
          <p class="ink-stagger-line ink-stagger-line--3 mx-auto mt-5 max-w-[820px] text-center font-medium" style="font-size: clamp(1.05rem, 1.75vw, 1.5rem); line-height: 1.38; color: color-mix(in srgb, var(--ink) 84%, transparent);">{{ brandDescription || t('home.hero.subtitle') }}</p>
          <div class="ink-stagger-line ink-stagger-line--4 mt-8 flex flex-wrap justify-center gap-3">
            <RouterLink to="/products" class="ink-btn ink-btn-primary ink-btn-knob ink-btn-lg">
              <span class="ink-knob"><Zap class="h-[20px] w-[20px]" /></span>
              {{ t('home.hero.cta') }}
            </RouterLink>
            <RouterLink to="/guest/orders" class="ink-btn ink-btn-ghost ink-btn-lg">{{ t('navbar.guestOrders') }}</RouterLink>
          </div>
        </div>
        <div class="ink-dot-rule mx-auto mt-12 max-w-[1080px]" aria-hidden="true"></div>
      </section>

      <!-- 分类块 -->
      <section v-if="topCategories.length" class="ink-section !py-9">
        <div class="mb-[22px] flex flex-wrap items-end justify-between gap-4">
          <h2 class="text-[28px] font-bold">{{ t('vault.categoriesTitle') }}</h2>
          <RouterLink to="/products" class="ink-btn ink-btn-ghost !h-9 !text-sm">{{ t('vault.allCategories') }} <ChevronRight class="h-4 w-4" /></RouterLink>
        </div>
        <div class="grid gap-3.5 grid-cols-[repeat(auto-fill,minmax(150px,1fr))]">
          <RouterLink
            v-for="cat in topCategories"
            :key="cat.id"
            class="flex min-h-[116px] flex-col justify-between gap-2.5 border bg-card p-[18px] transition-colors hover:border-[color:var(--ink-border-hover)]"
            style="border-color: var(--ink-border)"
            :to="`/categories/${cat.slug}`"
          >
            <span class="grid h-10 w-10 place-items-center bg-[color:var(--ink-warm)]">
              <img v-if="cat.icon" :src="getImageUrl(cat.icon)" :alt="catName(cat)" loading="lazy" class="h-6 w-6 object-contain" />
              <Tag v-else class="h-[22px] w-[22px]" />
            </span>
            <div class="min-w-0"><div class="line-clamp-2 break-words font-bold">{{ catName(cat) }}</div></div>
          </RouterLink>
        </div>
      </section>

      <!-- 热门商品 -->
      <section class="ink-section !pt-0">
        <div class="mb-[22px] flex flex-wrap items-end justify-between gap-4">
          <h2 class="text-[28px] font-bold">{{ t('home.featured.title') }}</h2>
          <RouterLink to="/products" class="ink-btn ink-btn-ghost !h-9 !text-sm">{{ t('home.featured.viewAll') }}</RouterLink>
        </div>

        <div v-if="productsLoading" class="grid gap-4 grid-cols-[repeat(auto-fill,minmax(228px,1fr))]">
          <div v-for="i in 10" :key="i" class="h-[280px] border bg-card" style="border-color: var(--ink-border)"></div>
        </div>
        <div v-else-if="products.length" class="grid gap-4 grid-cols-[repeat(auto-fill,minmax(228px,1fr))]">
          <InkProductCard
            v-for="(product, idx) in products"
            :key="product.id"
            :product="product"
            :index="idx"
            @quick-buy="openQuickBuy"
          />
        </div>
        <div v-else class="flex flex-col items-center gap-3 border border-dashed py-14 text-center text-muted-foreground" style="border-color: var(--ink-border)">
          <PackageOpen class="h-12 w-12 opacity-60" />
          <p>{{ t('home.featured.empty') }}</p>
        </div>
      </section>

      <!-- 最新动态 -->
      <section v-if="latestVisible && posts.length" class="ink-section !pt-0">
        <div class="mb-[22px] flex flex-wrap items-end justify-between gap-4">
          <h2 class="text-[28px] font-bold">{{ t('home.latest.title') }}</h2>
        </div>
        <div class="grid gap-4 grid-cols-[repeat(auto-fill,minmax(228px,1fr))]">
          <RouterLink
            v-for="post in posts"
            :key="post.id"
            class="ink-card block h-full"
            :to="`/blog/${post.slug}`"
          >
            <span class="text-[12.5px] font-semibold text-muted-foreground">{{ formatDate(post.published_at) }}</span>
            <h3 class="line-clamp-2 text-[17px] font-bold">{{ getLocalizedText(post.title) }}</h3>
            <p class="line-clamp-2 text-sm text-muted-foreground">{{ getLocalizedText(post.summary) }}</p>
            <span class="mt-auto inline-flex items-center gap-1.5 text-sm font-bold text-primary">{{ t('blog.readMore') }} <ChevronRight class="h-4 w-4" /></span>
          </RouterLink>
        </div>
      </section>
    </template>

    <ProductQuickBuy
      v-if="quickBuyProduct"
      :product="quickBuyProduct"
      :visible="quickBuyVisible"
      @update:visible="quickBuyVisible = $event"
    />

    <AnnouncementModal
      v-if="activeAnnouncement"
      :announcement="activeAnnouncement"
      :visible="announcementVisible"
      @update:visible="announcementVisible = $event"
    />
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { ChevronLeft, ChevronRight, PackageOpen, Search, SearchX, Tag, X, Zap } from 'lucide-vue-next'
import { categoryAPI, postAPI, productAPI } from '../../api'
import { buildCategoryGroups, type PublicCategory } from '../../utils/category'
import { getImageUrl } from '../../utils/image'
import { useLocalized } from '../../composables/useProduct'
import { useProductList } from '../../composables/useProductList'
import { useProductListGroups } from '../../composables/useProductListGroups'
import { usePageSeo } from '../../composables/usePageSeo'
import { useAppStore } from '../../stores/app'
import InkProductCard from './components/InkProductCard.vue'
import InkProductListItem from './components/InkProductListItem.vue'
import InkCategorySidebar from './components/InkCategorySidebar.vue'
import ProductQuickBuy from '../../components/ProductQuickBuy.vue'
import AnnouncementModal from '../../components/AnnouncementModal.vue'
import { useAnnouncement, type HomeAnnouncement } from '../../composables/useAnnouncement'

const route = useRoute()
const { t } = useI18n()
const { getLocalizedText } = useLocalized()
const appStore = useAppStore()

const isListMode = computed(() => appStore.config?.template_mode === 'list')

const brandName = computed(() => String(appStore.config?.brand?.site_name || '').trim())
const brandDescription = computed(() => {
  const desc = appStore.config?.brand?.site_description
  if (desc && typeof desc === 'object') {
    const val = (desc as Record<string, string>)[appStore.locale] || (desc as Record<string, string>)['zh-CN'] || ''
    return typeof val === 'string' ? val.trim() : ''
  }
  return ''
})

const quickBuyProduct = ref<any>(null)
const quickBuyVisible = ref(false)
const openQuickBuy = (product: any) => {
  quickBuyProduct.value = product
  quickBuyVisible.value = true
}

const {
  loading: listLoading,
  products: listProducts,
  selectedCategory,
  searchQuery,
  currentPage: listCurrentPage,
  totalPages: listTotalPages,
  expandedParentIds,
  categoryGroups,
  categoryMap: listCategoryMap,
  selectCategory,
  toggleParentCategory,
  changePage: listChangePage,
  clearSearch,
  initialize: listInitialize,
  cleanup: listCleanup,
} = useProductList({ pageSize: 20, homeRouteName: 'home' })

const listProductGroups = useProductListGroups(listProducts, listCategoryMap)

const listPageWindow = computed(() => {
  const total = listTotalPages.value
  const cur = listCurrentPage.value
  const start = Math.max(1, cur - 2)
  const end = Math.min(total, start + 4)
  const realStart = Math.max(1, end - 4)
  const pages: number[] = []
  for (let p = realStart; p <= end; p++) pages.push(p)
  return pages
})

const resetFilters = () => {
  clearSearch()
  selectCategory(null)
}

const products = ref<any[]>([])
const productsLoading = ref(true)
const posts = ref<any[]>([])
const topCategories = ref<PublicCategory[]>([])

const catName = (cat: PublicCategory) => getLocalizedText(cat.name) || cat.slug || ''

const navBuiltin = computed(() => (appStore.config?.nav_config as { builtin?: Record<string, boolean> } | undefined)?.builtin)
const blogEnabled = computed(() => navBuiltin.value?.blog !== false)
const noticeEnabled = computed(() => navBuiltin.value?.notice !== false)
const latestVisible = computed(() => blogEnabled.value || noticeEnabled.value)

const formatDate = (value: string) => (value ? new Date(value).toLocaleDateString() : '')

const { shouldShow } = useAnnouncement()
const activeAnnouncement = ref<HomeAnnouncement | null>(null)
const announcementVisible = ref(false)
const showAnnouncementIfNeeded = () => {
  const announcement = appStore.config?.announcement as HomeAnnouncement | undefined
  if (announcement && shouldShow(announcement)) {
    activeAnnouncement.value = announcement
    announcementVisible.value = true
  }
}

const loadProducts = async () => {
  productsLoading.value = true
  try {
    const res = await productAPI.list({ page: 1, page_size: 15 })
    products.value = res.data.data || []
  } catch (err) {
    console.error('Failed to load products:', err)
  } finally {
    productsLoading.value = false
  }
}

const loadCategories = async () => {
  try {
    const res = await categoryAPI.list()
    topCategories.value = buildCategoryGroups(res.data.data || []).slice(0, 10)
  } catch (err) {
    console.error('Failed to load categories:', err)
  }
}

const loadPosts = async () => {
  if (!latestVisible.value) return
  try {
    const params: Record<string, unknown> = { page: 1, page_size: 3 }
    if (blogEnabled.value && !noticeEnabled.value) params.type = 'blog'
    if (!blogEnabled.value && noticeEnabled.value) params.type = 'notice'
    const res = await postAPI.list(params)
    posts.value = res.data.data || []
  } catch (err) {
    console.error('Failed to load posts:', err)
  }
}

const seoCategoryName = computed(() => {
  if (!selectedCategory.value) return ''
  const cat = listCategoryMap.value.get(selectedCategory.value)
  return cat ? catName(cat) : ''
})
usePageSeo({
  canonicalPath: () => route.path,
  title: () => {
    if (route.name === 'category-products') return seoCategoryName.value || t('nav.products')
    if (route.name === 'products') return t('nav.products')
    return undefined
  },
})

onMounted(async () => {
  await appStore.loadConfig()
  if (isListMode.value) {
    await listInitialize()
  } else {
    await Promise.all([loadProducts(), loadCategories(), loadPosts()])
  }
  showAnnouncementIfNeeded()
})

onUnmounted(() => listCleanup())
</script>
