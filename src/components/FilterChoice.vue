<template>
  <button :class="{active: light}">
    {{ text }}
  </button>
</template>

<script>

export default {
    name: 'FilterChoice',
    props: {
        text: String,
        light: Boolean,
    },
}
</script>
