<template>
  <RouterLink :to="`/products/${product.slug}`" class="flex items-center gap-4 border bg-card px-4 py-3.5 transition-colors hover:border-[color:var(--ink-border-hover)]" style="border-color: var(--ink-border)" :class="{ 'opacity-[0.74]': soldOut }">
    <span class="relative grid h-14 w-14 flex-none place-items-center overflow-hidden bg-[color:var(--ink-warm)]">
      <img v-if="coverImage" :src="coverImage" :alt="title" loading="lazy" class="absolute inset-0 h-full w-full object-cover" @error="imageErrored = true" />
      <Package v-else class="relative z-[1] h-6 w-6 text-muted-foreground" />
    </span>
    <span class="min-w-0 flex-1">
      <span class="line-clamp-1 block font-bold">{{ title }}</span>
      <span v-if="categoryName" class="mt-0.5 block truncate text-xs text-muted-foreground">{{ categoryName }}</span>
    </span>
    <span class="flex-none text-right">
      <span class="block text-base font-bold tabular-nums">{{ formatPrice(promo ? getPromotionPriceAmount(product) : product.price_amount, siteCurrency) }}</span>
      <span class="mt-0.5 block text-xs font-semibold" :class="soldOut ? 'text-muted-foreground' : 'text-[color:var(--ink-success)]'">{{ getStockStatusLabel(product) }}</span>
    </span>
    <span v-if="!soldOut" class="ink-btn ink-btn-primary !h-9 flex-none !text-[13px] !font-semibold" @click.prevent.stop="$emit('quickBuy', product)">
      <Zap class="h-3.5 w-3.5" />
      {{ t('products.quickBuy') }}
    </span>
  </RouterLink>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import { useI18n } from 'vue-i18n'
import { Package, Zap } from 'lucide-vue-next'
import { getFirstImageUrl, getImageUrl } from '../../../utils/image'
import { useLocalized, useProductLabels } from '../../../composables/useProduct'

const props = defineProps<{ product: any }>()

defineEmits<{ quickBuy: [product: any] }>()

const { t } = useI18n()
const { getLocalizedText, siteCurrency, formatPrice } = useLocalized()
const { getStockStatusLabel, isSoldOut, hasPromotionPrice, getPromotionPriceAmount } = useProductLabels()

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
</script>
