<script setup lang="ts">
import type { GoogleSlidesProps } from "./GoogleSlides.types"
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from "vue"
import { cn } from "@/lib/utils"
import { Maximize2, Minimize2 } from "lucide-vue-next"
import { Button } from "@/components/ui"

const props = defineProps<GoogleSlidesProps>()

const containerRef = ref<HTMLElement | null>(null)
const iframeRef = ref<HTMLIFrameElement | null>(null)
const isCustomFullscreen = ref(false)
const isStandardFullscreen = ref(false)

/**
 * Check if we're in any fullscreen mode (standard or custom)
 */
const isFullscreen = computed(() => isStandardFullscreen.value || isCustomFullscreen.value)

/**
 * Toggle fullscreen mode with iOS Safari support
 */
const toggleFullscreen = () => {
  if (!containerRef.value) return

  // Check if browser supports standard Fullscreen API
  if (document.fullscreenElement) {
    document.exitFullscreen()
    return
  }

  // Try standard Fullscreen API first (works on most desktop browsers)
  if (containerRef.value.requestFullscreen) {
    containerRef.value.requestFullscreen().catch(() => {
      // Fallback to custom fullscreen if standard API fails
      toggleCustomFullscreen()
    })
  } else {
    // Use custom fullscreen for browsers that don't support Fullscreen API (like iOS Safari)
    toggleCustomFullscreen()
  }
}

/**
 * Custom fullscreen implementation for iOS and unsupported browsers
 */
const toggleCustomFullscreen = async () => {
  if (!containerRef.value) return

  isCustomFullscreen.value = !isCustomFullscreen.value

  if (isCustomFullscreen.value) {
    // Prevent body scrolling when in custom fullscreen
    document.body.style.overflow = 'hidden'

    // Wait for Vue to update the DOM
    await nextTick()

    // Force a reflow to ensure iframe resizes properly on mobile
    if (iframeRef.value) {
      // Read offsetHeight to force browser to recalculate layout
      void iframeRef.value.offsetHeight

      // Trigger window resize event to notify iframe of size change
      window.dispatchEvent(new Event('resize'))
    }
  } else {
    // Restore body scrolling
    document.body.style.overflow = ''
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

/**
 * Handle fullscreen change events for standard Fullscreen API
 */
const handleFullscreenChange = () => {
  isStandardFullscreen.value = !!document.fullscreenElement
}

/**
 * Set up event listeners for fullscreen changes
 */
onMounted(() => {
  document.addEventListener('fullscreenchange', handleFullscreenChange)
})

onBeforeUnmount(() => {
  document.removeEventListener('fullscreenchange', handleFullscreenChange)
})
</script>

<template>
  <div
    ref="containerRef"
    :class="cn(
      'w-full aspect-[16/10] rounded-lg overflow-hidden border border-border bg-muted relative group',
      isCustomFullscreen && 'fixed inset-0 z-[9999] aspect-auto rounded-none border-0',
      props.class
    )"
  >
    <iframe
      v-if="embedUrl"
      ref="iframeRef"
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
      :aria-label="isFullscreen ? 'Exit fullscreen' : 'Enter fullscreen'"
    >
      <Minimize2 v-if="isFullscreen" class="size-5" />
      <Maximize2 v-else class="size-5" />
    </Button>
  </div>
</template>
