<script setup lang="ts">
import type { GoogleSlidesProps } from "./GoogleSlides.types"
import { computed, ref } from "vue"
import { cn } from "@/lib/utils"
import { Maximize2 } from "lucide-vue-next"
import { Button } from "@/components/ui"

const props = defineProps<GoogleSlidesProps>()

const containerRef = ref<HTMLElement | null>(null)

/**
 * Toggle fullscreen mode
 */
const toggleFullscreen = () => {
  if (!containerRef.value) return

  if (!document.fullscreenElement) {
    containerRef.value.requestFullscreen()
  } else {
    document.exitFullscreen()
  }
}

/**
 * Extract presentation ID from various Google Slides URL formats
 */
const extractPresentationId = (url: string): { id: string; isPublished: boolean } | null => {
  // cspell:disable-next-line - PACX is part of Google's published presentation ID format
  // Match published presentation: /presentation/d/e/2PACX-{ID}/
  const publishedMatch = url.match(/\/presentation\/d\/e\/(2PACX-[a-zA-Z0-9-_]+)/)
  if (publishedMatch) {
    return { id: publishedMatch[1], isPublished: true }
  }

  // Match regular presentation: /presentation/d/{ID}/
  const regularMatch = url.match(/\/presentation\/d\/([a-zA-Z0-9-_]+)/)
  if (regularMatch) {
    return { id: regularMatch[1], isPublished: false }
  }

  return null
}

/**
 * Build embed URL with parameters (static: no autoplay, no loop)
 */
const embedUrl = computed(() => {
  const presentation = extractPresentationId(props.url)

  if (!presentation) {
    console.error("Invalid Google Slides URL:", props.url)
    return ""
  }

  const params = new URLSearchParams()
  params.append("start", "false")
  params.append("loop", "false")
  // cspell:disable-next-line - delayms is the correct Google Slides parameter
  params.append("delayms", "3000")

  const queryString = params.toString()

  // Build appropriate URL based on presentation type
  if (presentation.isPublished) {
    // Published presentation URL format
    return `https://docs.google.com/presentation/d/e/${presentation.id}/embed?${queryString}`
  } else {
    // Regular presentation URL format
    return `https://docs.google.com/presentation/d/${presentation.id}/embed?${queryString}`
  }
})
</script>

<template>
  <div
    ref="containerRef"
    :class="cn('w-full aspect-[16/10] rounded-lg overflow-hidden border border-border bg-muted relative group', props.class)"
  >
    <iframe
      v-if="embedUrl"
      :src="embedUrl"
      class="w-full h-full absolute inset-0"
      frameborder="0"
      allowfullscreen="true"
      mozallowfullscreen="true"
      webkitallowfullscreen="true"
      :title="`Google Slides Presentation`"
    />
    <!-- cspell:ignore mozallowfullscreen webkitallowfullscreen lucide - These are valid iframe attributes and library names -->
    <div
      v-else
      class="w-full h-full flex items-center justify-center text-muted-foreground"
    >
      <p>Invalid Google Slides URL</p>
    </div>

    <!-- Prominent Fullscreen Button -->
    <Button
      v-if="embedUrl"
      @click="toggleFullscreen"
      size="icon"
      variant="secondary"
      class="absolute bottom-4 right-4 z-50 opacity-100 md:opacity-0 md:group-hover:opacity-100 transition-opacity shadow-lg hover:shadow-xl pointer-events-auto"
      aria-label="Toggle fullscreen"
    >
      <Maximize2 class="size-5" />
    </Button>
  </div>
</template>
