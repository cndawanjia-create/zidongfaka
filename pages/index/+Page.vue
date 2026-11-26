<template>
  <main class="yy-page flex min-h-screen flex-col">
    <header class="fixed inset-x-0 top-0 z-50 border-b bg-background/90 backdrop-blur">
      <div class="mx-auto flex min-h-16 max-w-6xl items-center justify-between gap-4 px-5">
        <a href="/" class="flex min-w-0 items-center gap-2 font-semibold">
          <img :src="brandLogoUrl" :alt="`${site.name} Logo`" class="size-9 shrink-0 rounded-md object-contain" />
          <span class="truncate">{{ site.name }}</span>
        </a>
        <div class="flex shrink-0 items-center gap-2">
          <PublicNav />
        </div>
      </div>
    </header>

    <!-- 黑色公告条：模板样式，文字取后台真实公告 -->
    <section class="yy-notice-bar pt-16">
      <div v-if="localizedNotice" class="mx-auto flex max-w-6xl items-center gap-3 px-5 py-2 text-[13.5px]">
        <b class="shrink-0 font-bold text-amber-400">公告</b>
        <div class="min-w-0 overflow-hidden whitespace-nowrap">
          <p v-if="!shouldMarqueeNotice" class="m-0 truncate" :title="localizedNotice">{{ localizedNotice }}</p>
          <template v-else>
            <p class="sr-only">{{ localizedNotice }}</p>
            <div aria-hidden="true" class="notice-marquee-track inline-flex w-max animate-notice-marquee">
              <span class="shrink-0 pr-12">{{ localizedNotice }}</span>
              <span class="shrink-0 pr-12">{{ localizedNotice }}</span>
            </div>
          </template>
        </div>
      </div>
      <div v-else class="mx-auto flex max-w-6xl items-center gap-3 px-5 py-2 text-[13.5px]">
        <b class="shrink-0 font-bold text-amber-400">公告</b>
        <span class="truncate opacity-90">{{ localizedSubtitle || messages.storefront.fallbackSubtitle }}</span>
      </div>
    </section>

    <!-- Hero：模板样式，标题与副标题按银宇宙星球真实情况写 -->
    <section class="mx-auto w-full max-w-6xl px-5 pb-2 pt-8 sm:pt-10">
      <h1 class="max-w-3xl text-3xl font-black leading-tight tracking-tight sm:text-5xl">
        银宇宙星球账号商城，<span class="text-[#0e9f6e] dark:text-[#17b978]">下单即发</span>
      </h1>
      <p class="mt-4 max-w-2xl text-base text-muted-foreground">
        {{ localizedSubtitle || "人工检测后上架 · 付款后自动发货卡密 · 大批量采购请先联系客服谈批发价" }}
      </p>
      <form class="mt-6 flex max-w-2xl gap-2 rounded-full border bg-card p-1.5 shadow-lg" @submit.prevent>
        <div class="relative min-w-0 flex-1">
          <SearchIcon class="pointer-events-none absolute left-3 top-1/2 size-4 -translate-y-1/2 text-muted-foreground" />
          <Input v-model="query" class="border-0 bg-transparent pl-9 shadow-none focus-visible:ring-0" :placeholder="messages.storefront.searchProducts" :aria-label="messages.storefront.searchProducts" />
        </div>
        <Button type="submit" class="shrink-0 rounded-full bg-foreground px-6 text-background hover:bg-foreground/90">搜索</Button>
      </form>
      <div class="mt-5 flex flex-wrap items-center gap-2 text-[13px]">
        <span class="font-bold text-muted-foreground">付款方式</span>
        <span class="rounded-full bg-[#e3f5ec] px-3 py-1 font-semibold text-[#0e9f6e] dark:bg-[#123528] dark:text-[#17b978]">付款后自动发货</span>
        <span class="rounded-full border bg-card px-3 py-1 font-medium text-muted-foreground">支付宝</span>
        <span class="rounded-full border bg-card px-3 py-1 font-medium text-muted-foreground">微信</span>
        <span class="rounded-full border bg-card px-3 py-1 font-medium text-muted-foreground">USDT</span>
      </div>
      <div class="mt-7 flex flex-wrap gap-8">
        <div><b class="block text-2xl font-extrabold tabular-nums">{{ data.products.length }}</b><span class="text-[13px] text-muted-foreground">在售商品</span></div>
        <div><b class="block text-2xl font-extrabold tabular-nums">{{ data.categories.length }}</b><span class="text-[13px] text-muted-foreground">商品分类</span></div>
        <div><b class="block text-2xl font-extrabold tabular-nums">1</b><span class="text-[13px] text-muted-foreground">客服：Telegram</span></div>
      </div>
    </section>

    <!-- 商品区 -->
    <section id="shop" class="mx-auto w-full max-w-6xl scroll-mt-24 flex-1 px-5 py-8 sm:py-10">
      <div class="flex flex-wrap items-end justify-between gap-4">
        <div>
          <h2 class="text-2xl font-black tracking-normal">{{ messages.storefront.products }}</h2>
          <p class="mt-1 text-sm text-muted-foreground">{{ query ? t(messages.storefront.relatedProducts, { count: visibleProducts.length }) : messages.storefront.fallbackSubtitle }}</p>
        </div>
      </div>

      <div v-if="data.categories.length" class="mt-5 flex flex-wrap gap-2" :aria-label="messages.storefront.categories">
        <button
          type="button"
          class="inline-flex min-h-11 items-center gap-2 rounded-xl border-[1.5px] px-4 text-[15px] font-bold"
          :class="selectedCategory === null ? 'border-foreground bg-foreground text-background' : 'border-border bg-card hover:border-muted-foreground'"
          @click="selectedCategory = null"
        >
          {{ messages.storefront.all }}
          <span class="rounded-full bg-muted px-2 py-0.5 text-xs font-semibold text-muted-foreground">{{ data.products.length }}</span>
        </button>
        <button
          v-for="category in data.categories"
          :key="category.id"
          type="button"
          class="inline-flex min-h-11 items-center gap-2 rounded-xl border-[1.5px] px-4 text-[15px] font-bold"
          :class="selectedCategory === category.id ? 'border-foreground bg-foreground text-background' : 'border-border bg-card hover:border-muted-foreground'"
          @click="selectedCategory = category.id"
        >
          {{ categoryLabel(category) }}
          <span class="rounded-full bg-muted px-2 py-0.5 text-xs font-semibold text-muted-foreground">{{ categoryProductCount(category.id) }}</span>
        </button>
      </div>

      <div v-if="visibleProducts.length" class="mt-6 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
        <article v-for="product in visibleProducts" :key="product.id" class="group flex flex-col gap-3 overflow-hidden rounded-[14px] border bg-card p-4 transition-shadow hover:shadow-lg">
          <a :href="`/product/${product.slug}`" class="block">
            <div class="relative aspect-4/3 overflow-hidden rounded-[10px] bg-muted">
              <img :src="product.coverImage || defaultProductImage" :alt="productName(product)" class="size-full object-cover transition-transform duration-300 group-hover:scale-105" />
              <Badge variant="secondary" class="absolute left-2 top-2 bg-background/90 text-xs backdrop-blur">{{ productCategoryName(product) || messages.storefront.products }}</Badge>
            </div>
          </a>
          <a :href="`/product/${product.slug}`" class="block">
            <h3 class="line-clamp-2 text-[17px] font-bold leading-snug">{{ productName(product) }}</h3>
          </a>
          <p v-if="productSubtitle(product)" class="line-clamp-1 text-[13px] text-muted-foreground">{{ productSubtitle(product) }}</p>
          <div class="mt-auto flex flex-col gap-2.5 pt-1">
            <div class="flex items-end justify-between gap-2">
              <span class="text-2xl font-extrabold tabular-nums">{{ product.price }}<CurrencyHint /></span>
              <span :class="stockClass(product)" class="text-xs">{{ stockLabel(product) }}</span>
            </div>
            <a :href="`/product/${product.slug}`" class="yy-accent-btn block rounded-[10px] px-4 py-2.5 text-center text-[14.5px] font-extrabold">立即购买</a>
          </div>
        </article>
      </div>

      <div v-else class="mt-6 border border-dashed bg-background px-6 py-16 text-center">
        <PackageOpenIcon class="mx-auto size-8 text-muted-foreground" />
        <h3 class="mt-4 text-base font-semibold">{{ messages.storefront.noProducts }}</h3>
        <p class="mt-2 text-sm text-muted-foreground">{{ messages.storefront.noProductsDescription }}</p>
      </div>
    </section>

    <!-- 购买保障：只写已核实的政策（公告文案），不写模板的演示承诺 -->
    <section class="mx-auto w-full max-w-6xl px-5 pb-2 pt-6">
      <h2 class="text-2xl font-black">购买保障</h2>
      <p class="mt-1.5 text-[15px] text-muted-foreground">按本站实际政策执行，有问题请联系客服。</p>
      <div class="mt-5 grid gap-3 sm:grid-cols-2 lg:grid-cols-4">
        <div class="rounded-[14px] border bg-card p-5">
          <span class="text-[13px] font-extrabold tracking-widest text-[#0e9f6e] dark:text-[#17b978]">01</span>
          <b class="mt-2 block text-base">付款后自动发货</b>
          <p class="mt-1.5 text-sm text-muted-foreground">付款成功后系统自动发货卡密，在订单页查看交付内容。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="text-[13px] font-extrabold tracking-widest text-[#0e9f6e] dark:text-[#17b978]">02</span>
          <b class="mt-2 block text-base">上架前人工检测</b>
          <p class="mt-1.5 text-sm text-muted-foreground">全场账号均为人工检测后上架，付款后按订单顺序发货。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="text-[13px] font-extrabold tracking-widest text-[#0e9f6e] dark:text-[#17b978]">03</span>
          <b class="mt-2 block text-base">订单可查</b>
          <p class="mt-1.5 text-sm text-muted-foreground">下单后凭订单号在“我的订单”里查看订单状态和交付内容。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="text-[13px] font-extrabold tracking-widest text-[#0e9f6e] dark:text-[#17b978]">04</span>
          <b class="mt-2 block text-base">批发专属价</b>
          <p class="mt-1.5 text-sm text-muted-foreground">大批量采购请先联系客服谈批发价。</p>
        </div>
      </div>
    </section>

    <!-- 购买流程 -->
    <section id="steps" class="mx-auto w-full max-w-6xl scroll-mt-24 px-5 pb-2 pt-11">
      <h2 class="text-2xl font-black">购买流程</h2>
      <p class="mt-1.5 text-[15px] text-muted-foreground">四步完成，第一次买照着走就行。</p>
      <div class="mt-5 grid gap-3 sm:grid-cols-2 lg:grid-cols-4">
        <div class="rounded-[14px] border bg-card p-5">
          <span class="grid size-8 place-items-center rounded-full bg-foreground text-sm font-extrabold text-background">1</span>
          <b class="mt-3 block text-base">选择商品</b>
          <p class="mt-1.5 text-sm text-muted-foreground">在商品区按分类筛选，点进商品详情确认价格和库存。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="grid size-8 place-items-center rounded-full bg-foreground text-sm font-extrabold text-background">2</span>
          <b class="mt-3 block text-base">填写邮箱并付款</b>
          <p class="mt-1.5 text-sm text-muted-foreground">收货邮箱务必填对；支持支付宝、微信、USDT。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="grid size-8 place-items-center rounded-full bg-foreground text-sm font-extrabold text-background">3</span>
          <b class="mt-3 block text-base">查收卡密</b>
          <p class="mt-1.5 text-sm text-muted-foreground">付款成功后系统自动发货，在订单页查看账号、密码等交付内容。</p>
        </div>
        <div class="rounded-[14px] border bg-card p-5">
          <span class="grid size-8 place-items-center rounded-full bg-foreground text-sm font-extrabold text-background">4</span>
          <b class="mt-3 block text-base">登录验号</b>
          <p class="mt-1.5 text-sm text-muted-foreground">按商品说明登录并改绑自己的邮箱和手机号，有问题联系客服。</p>
        </div>
      </div>
    </section>

    <!-- 常见问题 -->
    <section id="faq" class="mx-auto w-full max-w-3xl scroll-mt-24 px-5 pb-2 pt-11">
      <h2 class="text-2xl font-black">常见问题</h2>
      <div class="mt-5 grid gap-2.5">
        <details class="rounded-xl border bg-card">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-3 px-5 py-3.5 font-bold">付款后怎么收到商品？<span class="shrink-0 text-xl text-muted-foreground">+</span></summary>
          <p class="px-5 pb-4 text-sm leading-6 text-muted-foreground">付款成功后系统自动发货，卡密会显示在订单页。没看到的话，先确认订单状态，再带订单号联系客服。</p>
        </details>
        <details class="rounded-xl border bg-card">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-3 px-5 py-3.5 font-bold">支持哪些付款方式？<span class="shrink-0 text-xl text-muted-foreground">+</span></summary>
          <p class="px-5 pb-4 text-sm leading-6 text-muted-foreground">支持支付宝、微信和 USDT（Arbitrum 网络）。下单页会显示当前可用的付款方式，以页面实际显示为准。</p>
        </details>
        <details class="rounded-xl border bg-card">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-3 px-5 py-3.5 font-bold">买回去登录不上怎么办？<span class="shrink-0 text-xl text-muted-foreground">+</span></summary>
          <p class="px-5 pb-4 text-sm leading-6 text-muted-foreground">先按商品说明检查账号、密码是否输对，以及是否需要改绑。仍有问题请带订单号联系客服，客服会按实际情况处理。</p>
        </details>
        <details class="rounded-xl border bg-card">
          <summary class="flex cursor-pointer list-none items-center justify-between gap-3 px-5 py-3.5 font-bold">可以大批量采购吗？<span class="shrink-0 text-xl text-muted-foreground">+</span></summary>
          <p class="px-5 pb-4 text-sm leading-6 text-muted-foreground">可以，大批量采购请先联系客服谈批发价，确认库存和价格后再下单。</p>
        </details>
      </div>
    </section>

    <!-- 售后与合规说明：用后台真实公告原文，不另编政策 -->
    <section class="mx-auto w-full max-w-6xl px-5 pt-8">
      <div class="rounded-[14px] border border-amber-500/60 bg-amber-50 p-5 dark:bg-amber-950/40">
        <b class="block">售后与合规说明</b>
        <p class="mt-1.5 text-sm leading-6 text-muted-foreground">{{ localizedNotice || "凡购买本站任何商品必须遵守各个国家法律法规及各大平台规则，非法用途一切后果自负，本站不承担任何法律及连带责任！" }}</p>
        <p class="mt-1.5 text-sm leading-6 text-muted-foreground">数字商品发货后即完成交付，请在收到后第一时间改绑邮箱、手机号并开启二次验证；后续因买家违规使用导致的问题，请联系客服按实际情况处理。</p>
      </div>
    </section>

    <div id="contact" class="scroll-mt-24">
      <StorefrontFooter />
    </div>

    <Dialog :open="noticePopupOpen" @update:open="closeNoticePopup">
      <DialogContent class="max-w-lg">
        <DialogHeader>
          <DialogTitle>{{ messages.storefront.noticeTitle }}</DialogTitle>
        </DialogHeader>
        <p class="whitespace-pre-wrap text-sm leading-6 text-muted-foreground">{{ localizedNotice }}</p>
        <DialogFooter>
          <DialogClose as-child><Button type="button" class="w-full">{{ messages.common.close }}</Button></DialogClose>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  </main>
</template>

<script lang="ts" setup>
import { computed, onMounted, ref } from "vue";
import { PackageOpenIcon, SearchIcon } from "@lucide/vue";
import logoUrl from "@/assets/logo.svg?url";
import defaultProductImage from "@/assets/product_img.jpg?url";
import { useData } from "vike-vue/useData";
import { usePageContext } from "vike-vue/usePageContext";
import PublicNav from "@/components/storefront/PublicNav.vue";
import StorefrontFooter from "@/components/storefront/StorefrontFooter.vue";
import { Badge } from "@/components/ui/badge";
import { Button } from "@/components/ui/button";
import { Dialog, DialogClose, DialogContent, DialogFooter, DialogHeader, DialogTitle } from "@/components/ui/dialog";
import { Input } from "@/components/ui/input";
import type { Data } from "./+data.server";
import CurrencyHint from "@/components/storefront/CurrencyHint.vue";
import { useStorefrontPreferences } from "@/lib/storefront-preferences";
import { pickLocalizedOptionalText, pickLocalizedText, type LocalizedNullableText } from "@/lib/i18n/localized-text";
import { readCookieValue } from "@/lib/i18n/locale";
import { noticeVersion } from "@/lib/notice-version";

type Product = Data["products"][number];
type Category = Data["categories"][number];
type Site = {
  name: string;
  subtitle: LocalizedNullableText;
  logo: string | null;
  notice: LocalizedNullableText;
  noticePopupEnabled: boolean;

  footerText: LocalizedNullableText;
};

const data = useData<Data>();
const pageContext = usePageContext() as unknown as { site: Site };
const site = pageContext.site;
const { messages, t, locale } = useStorefrontPreferences();
const selectedCategory = ref<number | null>(null);
const query = ref("");
const localizedNotice = computed(() => (site.notice ? pickLocalizedText(site.notice, locale.value) : ""));
const localizedSubtitle = computed(() => (site.subtitle ? pickLocalizedText(site.subtitle, locale.value) : ""));
const shouldMarqueeNotice = computed(() => localizedNotice.value.length > 30);
const noticeCookieName = "storefront-notice-seen";
const noticePopupOpen = ref(false);

function writeNoticeSeen() {
  const text = localizedNotice.value.trim();
  if (!text || typeof document === "undefined") return;
  const secure = window.location.protocol === "https:" ? "; secure" : "";
  document.cookie = `${noticeCookieName}=${noticeVersion(text)}; path=/; max-age=31536000; samesite=lax${secure}`;
}
function closeNoticePopup(open: boolean) {
  noticePopupOpen.value = open;
  if (!open) writeNoticeSeen();
}
onMounted(() => {
  const text = localizedNotice.value.trim();
  if (!site.noticePopupEnabled || !text) return;
  if (readCookieValue(document.cookie, noticeCookieName) !== noticeVersion(text)) noticePopupOpen.value = true;
});
const brandLogoUrl = computed(() => site.logo || logoUrl);
function productName(product: Product) { return pickLocalizedText(product.name, locale.value); }
function productSubtitle(product: Product) { return product.subtitle ? pickLocalizedOptionalText(product.subtitle, locale.value) : null; }
function productCategoryName(product: Product) { return product.categoryName ? pickLocalizedOptionalText(product.categoryName, locale.value) : null; }
function categoryLabel(category: Category) { return pickLocalizedText(category.name, locale.value); }
function categoryProductCount(categoryId: number) { return data.products.filter((product) => product.categoryId === categoryId).length; }
const visibleProducts = computed(() => {
  const keyword = query.value.trim().toLowerCase();
  return data.products.filter((product) =>
    (selectedCategory.value === null || product.categoryId === selectedCategory.value)
    && (!keyword || productName(product).toLowerCase().includes(keyword) || (productSubtitle(product) ?? "").toLowerCase().includes(keyword)),
  );
});

function stockLabel(product: Product) {
  if (product.availableStock === null) return messages.value.storefront.stockAvailable;
  return product.availableStock > 0
    ? t(messages.value.storefront.stockCount, { count: product.availableStock })
    : messages.value.storefront.outOfStock;
}
function stockClass(product: Product) {
  return product.availableStock === 0 ? "text-orange-500" : "text-muted-foreground";
}
</script>

<style scoped>
@keyframes notice-marquee {
  from {
    transform: translateX(0);
  }

  to {
    transform: translateX(-50%);
  }
}

.animate-notice-marquee {
  animation: notice-marquee 18s linear infinite;
}

@media (prefers-reduced-motion: reduce) {
  .animate-notice-marquee {
    animation: none;
  }
}
</style>
