<script setup>
const logout = async () => {
  navigateTo('/')
}
</script>

<template>
  <button @click="logout" class="text-sm text-red-500">Logout</button>
</template>