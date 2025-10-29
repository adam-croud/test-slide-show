<script setup lang="ts">
import Card from '@/components/ui/card.vue'
import CardHeader from '@/components/ui/card-header.vue'
import CardTitle from '@/components/ui/card-title.vue'
import CardDescription from '@/components/ui/card-description.vue'
import CardContent from '@/components/ui/card-content.vue'
import GoogleSlides from '@/components/GoogleSlides.vue'
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-blue-500 to-purple-600 p-4 md:p-8">
    <div class="max-w-6xl mx-auto space-y-8">
      <Card class="w-full">
        <CardHeader>
          <CardTitle>Welcome to Vue 3</CardTitle>
          <CardDescription>A minimal test setup with modern tooling</CardDescription>
        </CardHeader>
        <CardContent>
          <p class="text-sm mb-4">This project includes:</p>
          <ul class="text-sm space-y-2">
            <li>✓ Vue 3 + TypeScript</li>
            <li>✓ Vite</li>
            <li>✓ Tailwind CSS</li>
            <li>✓ Vue Router</li>
            <li>✓ shadcn/ui-vue</li>
            <li>✓ Google Slides Component</li>
          </ul>
        </CardContent>
      </Card>

      <Card class="w-full">
        <CardHeader>
          <CardTitle>Google Slides Demo</CardTitle>
          <CardDescription>Replace the url prop with your Google Slides presentation URL</CardDescription>
        </CardHeader>
        <CardContent>
          <GoogleSlides
            url="https://docs.google.com/presentation/d/e/2PACX-1vQouO1C9Wa5OA2uyyTiGjP6lZWNZHAT3f8mJvsNd_YpbRlB1xmo7_7qVXplZ0sI7ilZYIf9393w_8Dh/pub?start=false&loop=false&delayms=3000"
            class="w-full"
          />
        </CardContent>
      </Card>
    </div>
  </div>
</template>
