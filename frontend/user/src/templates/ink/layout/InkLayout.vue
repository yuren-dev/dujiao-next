<template>
  <div class="ink-scope">
    <header class="sticky top-0 z-50 border-b bg-[color:var(--ink-stage)]" style="border-color: var(--ink-border)">
      <div class="mx-auto flex h-[64px] w-full max-w-[1400px] items-center gap-3 px-[var(--ink-edge)]">
        <RouterLink class="inline-flex min-w-0 items-center gap-2.5 text-[19px] font-bold tracking-[-0.01em] text-foreground" to="/" :title="brandName">
          <img v-if="brandLogo" :src="brandLogo" :alt="brandName" class="h-8 max-w-[120px] object-contain sm:max-w-[160px]" />
          <span v-else class="truncate">{{ brandName }}</span>
        </RouterLink>

        <nav class="flex gap-0.5 max-[900px]:hidden">
          <template v-for="item in menuItems" :key="item.key">
            <RouterLink
              v-if="item.type === 'route'"
              :to="item.path"
              class="whitespace-nowrap px-3.5 py-2 text-[15px] font-medium text-muted-foreground transition-colors hover:text-foreground"
              active-class="!text-foreground"
            >{{ item.label }}</RouterLink>
            <a
              v-else
              :href="item.path"
              :target="item.target"
              rel="noopener noreferrer"
              class="whitespace-nowrap px-3.5 py-2 text-[15px] font-medium text-muted-foreground transition-colors hover:text-foreground"
            >{{ item.label }}</a>
          </template>
        </nav>

        <div class="ml-auto flex items-center gap-2">
          <RouterLink class="grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-foreground" to="/products" :aria-label="t('nav.products')"><Search class="h-[18px] w-[18px]" /></RouterLink>
          <RouterLink v-if="!userAuthStore.isAuthenticated" class="grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-foreground" to="/guest/orders" :aria-label="t('navbar.guestOrders')" :title="t('navbar.guestOrders')"><ClipboardList class="h-[18px] w-[18px]" /></RouterLink>
          <RouterLink class="relative grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-foreground" to="/cart" :aria-label="t('navbar.cart')">
            <ShoppingCart class="h-[18px] w-[18px]" />
            <span v-if="cartCount > 0" class="absolute -right-[3px] -top-[3px] grid h-[19px] min-w-[19px] place-items-center bg-primary px-[5px] text-[11px] font-bold text-white">{{ cartCount }}</span>
          </RouterLink>

          <div class="relative max-[900px]:hidden" ref="langEl">
            <button class="grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-foreground" type="button" :aria-label="t('navbar.selectLanguage')" @click="toggleLang">
              <Languages class="h-[18px] w-[18px]" />
            </button>
            <div v-if="langOpen" class="absolute right-0 top-[calc(100%+8px)] z-[60] flex min-w-[168px] flex-col gap-0.5 border bg-card p-2 shadow-[var(--ink-shadow-dialog)]" style="border-color: var(--ink-border)">
              <span class="px-3 pb-0.5 pt-1.5 text-[11px] font-bold uppercase tracking-[0.08em] text-muted-foreground">{{ t('navbar.selectLanguage') }}</span>
              <button v-for="lang in languages" :key="lang.code" class="flex w-full items-center justify-between gap-2.5 px-3 py-2.5 text-left text-sm font-medium transition-colors hover:text-foreground" :class="appStore.locale === lang.code ? 'text-primary' : 'text-muted-foreground'" @click="changeLanguage(lang.code)">
                {{ lang.name }}
                <span v-if="appStore.locale === lang.code" class="h-[7px] w-[7px] bg-primary"></span>
              </button>
            </div>
          </div>

          <template v-if="userAuthStore.isAuthenticated">
            <RouterLink class="ink-btn ink-btn-ghost !h-9 !text-[13px] !font-semibold max-[900px]:hidden" to="/me"><User class="h-[18px] w-[18px]" /> {{ t('navbar.personalCenter') }}</RouterLink>
            <button type="button" class="grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-destructive max-[900px]:hidden" :aria-label="t('navbar.logout')" :title="t('navbar.logout')" @click="userAuthStore.logout()"><LogOut class="h-[18px] w-[18px]" /></button>
          </template>
          <RouterLink v-else class="ink-btn ink-btn-primary !h-9 !text-[13px] !font-semibold max-[900px]:hidden" to="/auth/login">{{ t('navbar.login') }}</RouterLink>

          <div class="relative hidden max-[900px]:block" ref="moreEl">
            <button class="grid h-10 w-10 flex-none place-items-center text-muted-foreground transition-colors hover:text-foreground" type="button" :aria-label="t('navbar.more')" @click="toggleMore">
              <Menu v-if="!moreOpen" class="pointer-events-none h-[18px] w-[18px]" />
              <X v-else class="pointer-events-none h-[18px] w-[18px]" />
            </button>
            <div v-if="moreOpen" class="absolute right-0 top-[calc(100%+8px)] z-[60] flex min-w-[168px] flex-col gap-0.5 border bg-card p-2 shadow-[var(--ink-shadow-dialog)]" style="border-color: var(--ink-border)">
              <template v-for="item in menuItems" :key="`m-${item.key}`">
                <RouterLink v-if="item.type === 'route'" :to="item.path" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-sm font-medium text-muted-foreground hover:text-foreground" @click="moreOpen = false">{{ item.label }}</RouterLink>
                <a v-else :href="item.path" :target="item.target" rel="noopener noreferrer" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-sm font-medium text-muted-foreground hover:text-foreground" @click="moreOpen = false">{{ item.label }}</a>
              </template>
              <RouterLink v-if="!userAuthStore.isAuthenticated" to="/guest/orders" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-sm font-medium text-muted-foreground hover:text-foreground" @click="moreOpen = false">{{ t('navbar.guestOrders') }}</RouterLink>
              <RouterLink v-if="userAuthStore.isAuthenticated" to="/me" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-sm font-medium text-muted-foreground hover:text-foreground" @click="moreOpen = false">{{ t('navbar.personalCenter') }}</RouterLink>
              <RouterLink v-else to="/auth/login" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-sm font-medium text-muted-foreground hover:text-foreground" @click="moreOpen = false">{{ t('navbar.login') }}</RouterLink>
              <button v-if="userAuthStore.isAuthenticated" class="flex w-full items-center gap-2.5 px-3 py-2.5 text-left text-sm font-medium text-muted-foreground hover:text-foreground" @click="userAuthStore.logout(); moreOpen = false">{{ t('navbar.logout') }}</button>
              <span class="px-3 pb-0.5 pt-1.5 text-[11px] font-bold uppercase tracking-[0.08em] text-muted-foreground">{{ t('navbar.selectLanguage') }}</span>
              <button v-for="lang in languages" :key="`ml-${lang.code}`" class="flex w-full items-center justify-between gap-2.5 px-3 py-2.5 text-left text-sm font-medium transition-colors hover:text-foreground" :class="appStore.locale === lang.code ? 'text-primary' : 'text-muted-foreground'" @click="changeLanguage(lang.code); moreOpen = false">
                {{ lang.name }}
                <span v-if="appStore.locale === lang.code" class="h-[7px] w-[7px] bg-primary"></span>
              </button>
            </div>
          </div>
        </div>
      </div>
      <div class="ink-dot-rule" aria-hidden="true"></div>
    </header>

    <main class="flex-1">
      <slot />
    </main>

    <footer class="border-t bg-[color:var(--ink-warm)]" style="border-color: var(--ink-border)">
      <div class="mx-auto grid w-full max-w-[1400px] grid-cols-1 gap-[30px] px-[var(--ink-edge)] pb-9 pt-[52px] sm:grid-cols-2 lg:grid-cols-[1.7fr_repeat(3,1fr)]">
        <div>
          <RouterLink class="inline-flex items-center gap-2.5 text-[19px] font-bold tracking-[-0.01em] text-foreground" to="/">
            <img v-if="brandLogo" :src="brandLogo" :alt="brandName" class="h-8 max-w-[160px] object-contain" />
            <span v-else>{{ brandName }}</span>
          </RouterLink>
          <p class="mt-3 max-w-[36ch] text-[14.5px] text-muted-foreground">{{ brandDescription || t('vault.footer.tagline') }}</p>
        </div>
        <div>
          <h4 class="mb-3 text-sm font-bold">{{ t('vault.footer.shop') }}</h4>
          <RouterLink v-if="!isListMode" to="/products" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary">{{ t('products.allCategories') }}</RouterLink>
          <RouterLink v-if="noticeEnabled" to="/notice" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary">{{ t('nav.notice') }}</RouterLink>
          <RouterLink v-if="blogEnabled" to="/blog" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary">{{ t('nav.blog') }}</RouterLink>
          <RouterLink to="/me" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary">{{ t('navbar.personalCenter') }}</RouterLink>
        </div>
        <div>
          <h4 class="mb-3 text-sm font-bold">{{ t('vault.footer.support') }}</h4>
          <RouterLink v-if="aboutEnabled" to="/about" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><Info class="h-4 w-4" /> {{ t('nav.about') }}</RouterLink>
          <RouterLink v-if="!userAuthStore.isAuthenticated" to="/guest/orders" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><ClipboardList class="h-4 w-4" /> {{ t('navbar.guestOrders') }}</RouterLink>
          <a v-if="contact?.telegram" :href="contact.telegram" target="_blank" rel="noopener noreferrer" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><Send class="h-4 w-4" /> Telegram</a>
          <a v-if="contact?.whatsapp" :href="contact.whatsapp" target="_blank" rel="noopener noreferrer" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><MessageCircle class="h-4 w-4" /> WhatsApp</a>
          <a v-if="contact?.email" :href="`mailto:${contact.email}`" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><Mail class="h-4 w-4" /> {{ t('footer.email') }}</a>
          <a v-if="contact?.support_url" :href="contact.support_url" :target="contact.support_url.startsWith('https://') ? '_blank' : undefined" rel="noopener noreferrer" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary"><Headphones class="h-4 w-4" /> {{ t('footer.supportUrl') }}</a>
        </div>
        <div>
          <h4 class="mb-3 text-sm font-bold">{{ t('vault.footer.legal') }}</h4>
          <RouterLink v-for="link in footerLinks" :key="link.url" :to="link.url" class="flex items-center gap-[7px] py-[5px] text-[14.5px] text-muted-foreground hover:text-primary">{{ link.name }}</RouterLink>
        </div>
      </div>
    </footer>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import {
  ClipboardList, Headphones, Info, Languages, LayoutGrid, LogOut, Mail, Menu, MessageCircle,
  Search, Send, ShoppingCart, User, X,
} from 'lucide-vue-next'
import { useAppStore } from '../../../stores/app'
import { useCartStore } from '../../../stores/cart'
import { useUserAuthStore } from '../../../stores/userAuth'
import { useNavConfig, type NavItem } from '../../../composables/useNavConfig'
import { getImageUrl } from '../../../utils/image'
import { getLocalizedText } from '../../../utils/resellerSiteConfig'
import '../styles/ink.css'

const { t } = useI18n()
const appStore = useAppStore()
const cartStore = useCartStore()
const userAuthStore = useUserAuthStore()
const { isListMode, blogEnabled, noticeEnabled, aboutEnabled, secondaryNavItems } = useNavConfig()

const langEl = ref<HTMLElement | null>(null)
const moreEl = ref<HTMLElement | null>(null)
const langOpen = ref(false)
const moreOpen = ref(false)

const brandName = computed(() => String(appStore.config?.brand?.site_name || '').trim())
const brandLogo = computed(() => {
  const raw = String(appStore.config?.brand?.site_logo || '').trim()
  return raw ? getImageUrl(raw) : ''
})
const brandDescription = computed(() => {
  const desc = appStore.config?.brand?.site_description
  if (desc && typeof desc === 'object') {
    const val = (desc as Record<string, string>)[appStore.locale] || (desc as Record<string, string>)['zh-CN'] || ''
    return typeof val === 'string' ? val.trim() : ''
  }
  return ''
})

const menuItems = computed<NavItem[]>(() => {
  const items: NavItem[] = []
  if (!isListMode.value) {
    items.push({ key: 'products', path: '/products', label: t('products.allCategories'), icon: LayoutGrid, type: 'route', target: '_self' })
  }
  items.push(...secondaryNavItems.value)
  return items
})

const footerLinks = computed(() => {
  const links = appStore.config?.footer_links
  if (!Array.isArray(links)) return []
  return links
    .map((item: { name?: unknown; url?: unknown }) => ({
      name: typeof item?.name === 'string' ? item.name.trim() : getLocalizedText(item?.name as Record<string, string>, appStore.locale),
      url: String(item?.url || '').trim(),
    }))
    .filter((item) => item.name)
})

const contact = computed(() => appStore.config?.contact as { telegram?: string; whatsapp?: string; email?: string; support_url?: string } | undefined)
const cartCount = computed(() => cartStore.totalItems)

const languages = [
  { code: 'zh-CN', name: '简体中文' },
  { code: 'zh-TW', name: '繁體中文' },
  { code: 'en-US', name: 'English' },
]
const changeLanguage = (code: string) => {
  appStore.setLocale(code)
  langOpen.value = false
}
const toggleLang = () => { langOpen.value = !langOpen.value; moreOpen.value = false }
const toggleMore = () => { moreOpen.value = !moreOpen.value; langOpen.value = false }
const onDocClick = (e: MouseEvent) => {
  const target = e.target as Node
  if (langOpen.value && langEl.value && !langEl.value.contains(target)) langOpen.value = false
  if (moreOpen.value && moreEl.value && !moreEl.value.contains(target)) moreOpen.value = false
}

onMounted(() => {
  document.addEventListener('click', onDocClick)
  document.body.classList.add('ink-tokens')
})
onUnmounted(() => {
  document.removeEventListener('click', onDocClick)
  document.body.classList.remove('ink-tokens')
})
</script>
