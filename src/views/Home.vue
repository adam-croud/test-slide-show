<script setup lang="ts">
import Card from '@/components/ui/card.vue'
import CardHeader from '@/components/ui/card-header.vue'
import CardTitle from '@/components/ui/card-title.vue'
import CardDescription from '@/components/ui/card-description.vue'
import CardContent from '@/components/ui/card-content.vue'
</script>

<template>
  <div class="min-h-screen flex items-center justify-center bg-gradient-to-br from-blue-500 to-purple-600 p-4">
    <Card class="w-full max-w-md">
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
        </ul>
      </CardContent>
    </Card>
  </div>
</template>
