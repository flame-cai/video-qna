<template>
  <div class="loading-component">
    <h2>Your questions are being generated...</h2>
    <div class="loader"></div>
  </div>
</template>

<style>
div.loading-component {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  flex-grow: 1;
}

.loader {
  margin-top: 3em;
  width: 40px;
  aspect-ratio: 1;
  color: orange;
  background: repeating-conic-gradient(
    from -47deg,
    #0000 0deg,
    currentColor 1deg 91deg,
    #0000 94deg 180deg
  );
  display: flex;
  animation: l12-0 2s infinite linear;
}

.loader::before,
.loader::after {
  content: '';
  flex: 1;
  background: currentColor;
  clip-path: polygon(0 0, 100% 50%, 0 100%);
  animation: l12 1s infinite alternate;
  transform-origin: bottom left;
}

.loader::after {
  clip-path: polygon(100% 0, 100% 100%, 0 50%);
  transform-origin: top right;
}

@keyframes l12-0 {
  0%,
  49.9% {
    transform: scaleX(1);
  }
  50%,
  100% {
    transform: scaleX(-1);
  }
}

@keyframes l12 {
  0%,
  20% {
    transform: rotate(0deg);
  }
  80%,
  100% {
    transform: rotate(-270deg);
  }
}
</style>
