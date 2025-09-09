<script setup>
const props = defineProps({ clerk: Object });

function handleLogout() {
  if (props.clerk.session) {
    props.clerk.session.end().then(() => {
      // Nach erfolgreichem Logout Seite neu laden
      window.location.reload();
    });
  } else {
    window.location.reload();
  }
}
</script>

<template>
  <button @click="handleLogout">
    Logout
  </button>
</template>

<style scoped>
button {
  background: #e53e3e;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
button:hover {
  background: #c53030;
}
</style>
