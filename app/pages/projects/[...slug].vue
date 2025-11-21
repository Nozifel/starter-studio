
<script lang="ts" setup>
import type { Collections } from '@nuxt/content'
import { withLeadingSlash, joinURL } from 'ufo'

const route = useRoute()
const { locale, t, localeProperties } = useI18n()

const slug = computed(() => Array.isArray(route.params.slug) ? route.params.slug as string[] : [route.params.slug as string])
const path = computed(() => withLeadingSlash(joinURL(locale.value, 'projects', ...slug.value)))
const collection = computed(() => `projects_${locale.value}` as keyof Collections)

const { data: page } = await useAsyncData(path.value, async () =>
  await queryCollection(collection.value).path(path.value).first() as Collections['projects_en'] | Collections['projects_fr'],
)

if (!page.value)
  throw createError({ statusCode: 404, statusMessage: 'Page not found' })

const { copy } = useClipboard()

function copyArticleLink() {
  copy(`${window.location.origin}${route.fullPath}`)
  toast.success(t('global.article_link_copied'))
}

defineShortcuts({
  meta_k: {
    usingInput: true,
    handler: () => {
      copy(`${window.location.origin}${route.fullPath}`)
      toast.success(t('global.article_link_copied'))
    },
  },
})

defineOgImage({
  url: page.value.image,
})
</script>

<template>
  <div v-if="page">
      <ContentRenderer
        v-if="page"
        :dir="localeProperties?.dir ?? 'ltr'"
        :value="page"
      />
    </article>
  </div>
</template>

<style scoped>
.info-section {
  font-weight: 200;
  color: #7d8084;
  text-decoration: none;
  text-align: left;
}
</style>
