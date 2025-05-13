<template>
  <VApp>
    <NuxtLayout>
      <NuxtPage />
      <div
        v-if="!isLoadDone"
        class="loading-page d-flex flex-row justify-center align-center"
      >
        <div
          class="d-flex logofull"
          v-html="logofull"
        />
      </div>
    </NuxtLayout>
    <GeneralDialog />
    <GeneralToast />
  </VApp>
</template>
