<template>
  <footer
    class="absolute bottom-0 flex items-center justify-end w-full h-10 py-2 pr-8 text-sm font-normal text-gray-700 bg-white"
  >
    {{ $t('general.powered_by') }}

    <a
      href="https://app.telcoges.com/"
      target="_blank"
      class="pl-1 font-normal text-gray-900"
    >
      {{ $t('general.telcoges') }}
    </a>
  </footer>
</template>
