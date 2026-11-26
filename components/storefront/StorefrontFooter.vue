<template>
  <footer class="mt-14 border-t bg-background/80">
    <div class="mx-auto grid max-w-6xl gap-7 px-5 py-9 text-sm sm:grid-cols-[1.4fr_1fr_1fr]">
      <div>
        <h4 class="text-[15px] font-bold">{{ siteName }}</h4>
        <p class="mt-3 whitespace-pre-wrap leading-6 text-muted-foreground">{{ footerText }}</p>
      </div>
      <div>
        <h4 class="text-[15px] font-bold">快速导航</h4>
        <ul class="mt-3 grid gap-2 text-muted-foreground">
          <li><a href="/#shop" class="hover:text-foreground hover:underline">全部商品</a></li>
          <li><a href="/#steps" class="hover:text-foreground hover:underline">购买流程</a></li>
          <li><a href="/#faq" class="hover:text-foreground hover:underline">常见问题</a></li>
          <li><a href="/order" class="hover:text-foreground hover:underline">订单查询</a></li>
        </ul>
      </div>
      <div>
        <h4 class="text-[15px] font-bold">联系客服</h4>
        <ul class="mt-3 grid gap-2 text-muted-foreground">
          <li v-for="contact in supportContacts" :key="`${contact.label}-${contact.href}`">
            <a :href="contact.href" class="hover:text-foreground hover:underline" target="_blank" rel="noopener noreferrer">{{ contact.label }}</a>
          </li>
          <li v-if="!supportContacts.length">Telegram：@Yinyuzhou</li>
        </ul>
      </div>
    </div>
    <div class="border-t">
      <div class="mx-auto flex max-w-6xl flex-wrap items-center justify-between gap-2 px-5 py-4 text-[13px] text-muted-foreground">
        <span>© 2026 {{ siteName }}</span>
        <span>付款后自动发货 · 人工检测后上架</span>
      </div>
    </div>
  </footer>
</template>

<script lang="ts" setup>
import { computed } from "vue";
import { usePageContext } from "vike-vue/usePageContext";

import { useStorefrontPreferences } from "@/lib/storefront-preferences";
import { pickLocalizedText, type LocalizedText } from "@/lib/i18n/localized-text";

type Site = {
  name: string;
  supportContact: LocalizedText;
  footerText: LocalizedText;
};

type SupportContact = {
  label: string;
  href: string;
};

const pageContext = usePageContext() as unknown as { site: Site };
const site = pageContext.site;
const { locale } = useStorefrontPreferences();
const siteName = computed(() => site.name || "银宇宙星球");
const footerText = computed(() => pickLocalizedText(site.footerText, locale.value));
const supportContacts = computed<SupportContact[]>(() => pickLocalizedText(site.supportContact, locale.value)
  .split("\n")
  .map((line) => line.trim())
  .filter(Boolean)
  .map((line) => {
    const separatorIndex = line.indexOf("|");
    const label = separatorIndex >= 0 ? line.slice(0, separatorIndex).trim() : line;
    const address = separatorIndex >= 0 ? line.slice(separatorIndex + 1).trim() : line;
    if (!label || !address) return null;
    return { label, href: address.includes(":") ? address : `mailto:${address}` };
  })
  .filter((contact): contact is SupportContact => contact !== null));
</script>
