<template>
  <RouterLink :to="`/products/${product.slug}`" class="ink-card h-full text-left" :class="{ 'opacity-[0.74]': soldOut }">
    <span
      class="relative grid h-[152px] w-full flex-none place-items-center overflow-hidden bg-[color:var(--ink-warm)]"
      style="border: 1px solid var(--ink-border)"
    >
      <img v-if="coverImage" :src="coverImage" :alt="title" loading="lazy" class="absolute inset-0 h-full w-full object-cover" @error="imageErrored = true" />
      <Package v-else class="relative z-[1] h-[58px] w-[58px] text-muted-foreground" />
      <div v-if="!soldOut && product.tags && product.tags.length" class="absolute right-1.5 top-1.5 z-[3] flex max-w-[80%] flex-wrap justify-end gap-1">
        <span v-for="(tag, i) in product.tags.slice(0, 2)" :key="i" class="inline-flex max-w-full truncate bg-black/55 px-2 py-0.5 text-[11px] font-semibold text-white">{{ tag }}</span>
      </div>
    </span>

    <div class="flex min-w-0 flex-col gap-0.5 px-1">
      <span v-if="categoryName" class="truncate text-xs font-semibold text-muted-foreground">{{ categoryName }}</span>
      <h3 class="line-clamp-2 text-base font-bold leading-[1.3]">{{ title }}</h3>
    </div>

    <div class="mx-1 flex flex-wrap items-center gap-1.5">
      <span class="inline-flex items-center gap-1 bg-[color:var(--ink-success-soft)] px-2 py-0.5 text-[11px] font-semibold text-[color:var(--ink-success)]">
        <component :is="product.fulfillment_type === 'auto' ? Zap : Pencil" class="h-3 w-3" />
        {{ getFulfillmentTypeLabel(product.fulfillment_type) }}
      </span>
      <span class="inline-flex items-center gap-1 px-2 py-0.5 text-[11px] font-semibold" :class="product.purchase_type === 'guest' ? 'bg-[color:var(--ink-amber-soft)] text-[color:var(--ink-amber-deep)]' : 'bg-[color:var(--ink-success-soft)] text-[color:var(--ink-success)]'">
        <component :is="product.purchase_type === 'guest' ? UserPlus : Lock" class="h-3 w-3" />
        {{ getPurchaseTypeLabel(product.purchase_type) }}
      </span>
      <span class="inline-flex items-center gap-1 px-2 py-0.5 text-[11px] font-semibold" :class="stockPill.tone">
        <component :is="stockPill.icon" class="h-3 w-3" />
        {{ stockPill.label }}
      </span>
    </div>

    <div class="mt-auto flex items-end justify-between gap-2 px-1">
      <div class="flex min-w-0 flex-col gap-1">
        <div class="flex flex-wrap items-baseline gap-x-1.5">
          <template v-if="promo">
            <span class="text-xl font-bold tabular-nums text-primary">{{ formatPrice(getPromotionPriceAmount(product), siteCurrency) }}</span>
            <span class="text-sm font-semibold text-muted-foreground line-through">{{ formatPrice(product.price_amount, siteCurrency) }}</span>
          </template>
          <span v-else class="text-xl font-bold tabular-nums text-foreground">{{ formatPrice(product.price_amount, siteCurrency) }}</span>
        </div>
        <span v-if="priceSignal" class="inline-flex w-fit px-2 py-0.5 text-[11px] font-semibold" :class="priceSignal.tone">{{ priceSignal.label }}</span>
      </div>
      <span v-if="soldOut" class="ink-btn ink-btn-ghost !h-9 flex-none !text-[13px] !font-semibold" aria-disabled="true">{{ t('products.stockStatus.outOfStock') }}</span>
      <button
        v-else
        type="button"
        class="ink-btn ink-btn-primary !h-9 flex-none !text-[13px] !font-semibold"
        :aria-label="t('products.quickBuyAria')"
        @click.prevent.stop="$emit('quickBuy', product)"
      >
        <Zap class="h-3.5 w-3.5" />
        {{ t('products.quickBuy') }}
      </button>
    </div>
  </RouterLink>
</template>

<script setup lang="ts">
import { computed, ref, type Component } from 'vue'
import { useI18n } from 'vue-i18n'
import { AlarmClock, Lock, Package, Pencil, UserPlus, XCircle, Zap } from 'lucide-vue-next'
import { getFirstImageUrl, getImageUrl } from '../../../utils/image'
import { useLocalized, useProductLabels } from '../../../composables/useProduct'

const props = withDefaults(defineProps<{ product: any; index?: number }>(), { index: 0 })

defineEmits<{ quickBuy: [product: any] }>()

const { t } = useI18n()
const { getLocalizedText, siteCurrency, formatPrice } = useLocalized()
const {
  getStockStatusLabel, getPurchaseTypeLabel, getFulfillmentTypeLabel,
  isSoldOut, hasPromotionPrice, getPromotionPriceAmount, hasWholesalePrices, hasPromotionRules,
} = useProductLabels()

const title = computed(() => getLocalizedText(props.product?.title))
const categoryName = computed(() => getLocalizedText(props.product?.category?.name))
const soldOut = computed(() => isSoldOut(props.product))
const promo = computed(() => hasPromotionPrice(props.product))

const imageErrored = ref(false)
const coverImage = computed(() => {
  if (imageErrored.value) return ''
  const primary = getFirstImageUrl(props.product?.images)
  if (primary) return primary
  const icon = props.product?.category?.icon
  return icon ? getImageUrl(icon) : ''
})

const stockPill = computed<{ tone: string; icon: Component; label: string }>(() => {
  if (soldOut.value) {
    return { tone: 'bg-secondary text-muted-foreground', icon: XCircle, label: t('products.stockStatus.outOfStock') }
  }
  if (props.product?.stock_status === 'low_stock') {
    return { tone: 'bg-[color:var(--ink-amber-soft)] text-[color:var(--ink-amber-deep)]', icon: AlarmClock, label: getStockStatusLabel(props.product) }
  }
  return { tone: 'bg-[color:var(--ink-success-soft)] text-[color:var(--ink-success)]', icon: Zap, label: getStockStatusLabel(props.product) }
})

const priceSignal = computed<{ tone: string; label: string } | null>(() => {
  if (promo.value) return { tone: 'bg-primary/10 text-primary', label: t('products.promotionTag') }
  if (hasWholesalePrices(props.product)) return { tone: 'bg-[color:var(--ink-success-soft)] text-[color:var(--ink-success)]', label: t('products.wholesaleTag') }
  if (hasPromotionRules(props.product)) return { tone: 'bg-[color:var(--ink-amber-soft)] text-[color:var(--ink-amber-deep)]', label: t('products.promotionBadge') }
  return null
})
</script>
