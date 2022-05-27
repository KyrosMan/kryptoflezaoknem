<template>
<div class="veil">
    <div class="lds-dual-ring"></div>
    <div> {{text}} </div>
</div>
    
</template>

<script>

export default {
  name: "LoaderEl",
  props: {
    text: String
  }
}
</script>

<style scoped>
.lds-dual-ring {
  display: inline-block;
  width: 80px;
  height: 80px;
}
.lds-dual-ring:after {
  content: " ";
  display: block;
  width: 64px;
  height: 64px;
  margin: 8px;
  border-radius: 50%;
  border: 6px solid #fff;
  border-color: #fff transparent #fff transparent;
  animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.veil {
    height: 100vh;
    position: absolute;
    top: 0;
    left: 0;
    width: 100vw;

    display: flex;
    gap: 50px;
    align-items: center;
    justify-content: center;
    flex-direction: column;
    z-index: 10;
    background-color: black;
    opacity: 0.5;
}
</style>