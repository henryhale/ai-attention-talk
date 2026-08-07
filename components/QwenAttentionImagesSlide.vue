<template>
  <div class="slide-shell qwen-attention-slide">
    <h1>Qwen3-8B uses grouped-query attention.</h1>

    <section class="attention-image-stage" aria-live="polite">
      <img
        v-motion
        :initial="{ opacity: 1 }"
        :click-1="{ opacity: 0 }"
        class="attention-image image-first"
        src="/mha-mqa-conversion.webp"
        alt="Multi-head attention, multi-query attention, and conversion from MHA to MQA"
      >
      <img
        v-click="1"
        class="attention-image image-second"
        src="/mha-mqa.webp"
        alt="Comparison of multi-head, grouped-query, and multi-query attention"
      >
    </section>

    <footer class="caption-stage">
      <p
        v-motion
        :initial="{ opacity: 1 }"
        :click-1="{ opacity: 0 }"
      >
        Left — Multihead attention, Middle — MultiQuery Attention, Right — Converting existing MHA checkpoint to MQA (Source — <a href="https://arxiv.org/pdf/2305.13245.pdf" target="_blank" rel="noopener noreferrer">https://arxiv.org/pdf/2305.13245.pdf</a>)
      </p>
      <p v-click="1">
        Difference between MHA, GQA, and MQA (Source — <a href="https://arxiv.org/pdf/2305.13245.pdf" target="_blank" rel="noopener noreferrer">https://arxiv.org/pdf/2305.13245.pdf</a>)
      </p>
    </footer>
  </div>
</template>

<style scoped>
.qwen-attention-slide {
  padding: 27px 38px 20px;
}

.qwen-attention-slide h1 {
  flex: 0 0 auto;
  padding-bottom: 13px;
  border-bottom: 1px solid #292e33;
  font-size: 34px;
  line-height: 1;
  text-align: center;
}

.attention-image-stage {
  position: relative;
  flex: 1 1 auto;
  min-height: 0;
  margin: 14px 0 10px;
}

.attention-image {
  position: absolute;
  inset: 0;
  display: block;
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 8px;
  filter: drop-shadow(0 14px 30px rgba(0, 0, 0, 0.28));
}

.image-second.slidev-vclick-target {
  transition: opacity 0.55s ease;
}

.caption-stage {
  position: relative;
  flex: 0 0 62px;
  border-top: 1px solid #292e33;
}

.caption-stage p {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  padding: 8px 16px 0;
  color: #8e999e;
  font-size: 10.5px;
  font-weight: 500;
  line-height: 1.45;
  text-align: center;
}

.caption-stage p.slidev-vclick-target {
  transition: opacity 0.55s ease;
}

.caption-stage a {
  margin-left: 3px;
  color: var(--accent);
  text-decoration: none;
}
</style>
