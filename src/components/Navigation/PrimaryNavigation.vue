<template>
  <div class="h-0 flex-1 flex flex-col pb-4 overflow-y-auto">
    <div
      class="flex-shrink-0 flex items-center px-4 py-3 border-b border-gray-200"
    >
      <h1>COVID-19 Dashboard</h1>
    </div>
    <nav class="mt-3 px-2">
      <button
        class="text-gray-900 bg-gray-100 focus:outline-none focus:bg-gray-200"
      >
        Timeline
      </button>
      <button
        class="mt-1 text-gray-600 hover:text-gray-900 hover:bg-gray-50 focus:outline-none focus:text-gray-900 focus:bg-gray-100"
      >
        Suppression Efficacy
      </button>
    </nav>
  </div>
</template>

<style scoped>
button {
  @apply flex w-full items-center px-2 py-2 text-base leading-6 font-medium rounded-md transition ease-in-out duration-150;
}
</style>
