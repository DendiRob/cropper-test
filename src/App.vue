<template>
  <div class="cropper">
    <input type="file" name="image" accept="image/*" @change="onAddFile" />

    <div class="cropper__contaienr" v-if="imageObjectUrl">
      {{ cropBoxStyles }}
      <div class="crop__box" :style="cropBoxStyles">
        <img :src="imageObjectUrl" alt="img" :style="imageStyles" />
        <div
          class="angle top-left"
          @mousedown.stop="(me) => handleResizeAngle('tl', me)"
        ></div>
        <div
          class="angle top-right"
          @mousedown.stop="(me) => handleResizeAngle('tr', me)"
        ></div>
        <div
          class="angle bottom-left"
          @mousedown.stop="(me) => handleResizeAngle('bl', me)"
        ></div>
        <div
          class="angle bottom-right"
          @mousedown.stop="(me) => handleResizeAngle('br', me)"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, CSSProperties, ref } from "vue";

type TResizedAngle = "tl" | "tr" | "bl" | "br" | null;

const imageObjectUrl = ref();
const resizedAngle = ref<TResizedAngle>(null);
const onAddFile = (e: Event) => {
  const target = e.target as HTMLInputElement;
  const file = target.files?.[0];
  if (file) imageObjectUrl.value = URL.createObjectURL(file);
};

const crop = ref({ x: 0, y: 0, width: 700, height: 500 });
const startCrop = ref({ x: 0, y: 0, width: 0, height: 0 });
const startMouse = ref({ x: 0, y: 0 });

const dragging = ref(false);

const cropBoxStyles = computed<CSSProperties>(() => ({
  top: `${crop.value.y}px`,
  left: `${crop.value.y}px`,
  width: `${crop.value.width}px`,
  height: `${crop.value.height}px`,
  // overflow: "hidden",
  position: "absolute",
}));

const imageStyles = computed<CSSProperties>(() => ({
  top: `-${crop.value.y}px`,
  left: `-${crop.value.y}px`,
  objectFit: "contain",
  maxHeight: "500px",
  maxWidth: "700px",
  position: "absolute",
}));

const onResize = (e: MouseEvent) => {
  if (!resizedAngle.value) return;
  const dx = e.clientX - startMouse.value.x;
  const dy = e.clientY - startMouse.value.y;

  console.log(dx, dy);

  switch (resizedAngle.value) {
    case "tl":
      crop.value.width = startCrop.value.width - dx;
      crop.value.height = startCrop.value.height - dy;

      crop.value.x = startCrop.value.x + dx;
      crop.value.y = startCrop.value.y + dy;

      return;
    case "bl":
      // crop.value.width = startCrop.value.width - dx;
      // crop.value.height = startCrop.value.height + dy;
      // crop.value.x = startCrop.value.x + dx;
      return;
    case "br":
      crop.value.width = startCrop.value.width + dx;
      crop.value.height = startCrop.value.height + dy;
      return;
    case "tr":
      crop.value.width = startCrop.value.width + dx;
      crop.value.height = startCrop.value.height - dy;
      crop.value.y = startCrop.value.y + dy;
      return;
  }
};

const stopResize = () => {
  resizedAngle.value = null;
  window.removeEventListener("mousemove", onResize);
  window.removeEventListener("mouseup", onResize);
};

const handleResizeAngle = (e: TResizedAngle, me: MouseEvent) => {
  resizedAngle.value = e;
  startCrop.value.x = crop.value.x;
  startCrop.value.y = crop.value.y;
  startCrop.value.width = crop.value.width;
  startCrop.value.height = crop.value.height;
  startMouse.value.x = me.clientX;
  startMouse.value.x = me.clientY;

  window.addEventListener("mousemove", onResize);
  window.addEventListener("mouseup", stopResize);
};
</script>

<style scoped>
.cropper {
  width: 700px;
  height: 500px;
  position: relative;
}

.crop__box {
  border: 2px solid green;
  cursor: move;
  position: relative;
}

.angle {
  width: 10px;
  height: 10px;
  cursor: pointer;
  border-radius: 100%;
  background-color: brown;
  position: absolute;
  z-index: 2;
}

.angle.top-left {
  top: -5px;
  left: -5px;
  cursor: grab;
}

.angle.top-right {
  top: -5px;
  right: -5px;
  cursor: grab;
}

.angle.bottom-right {
  bottom: -5px;
  right: -5px;
  cursor: grab;
}

.angle.bottom-left {
  bottom: -5px;
  left: -5px;
  cursor: grab;
}
</style>
