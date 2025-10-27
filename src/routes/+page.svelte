<script lang="ts">
  import { onMount } from "svelte";
  import ColorPicker from "svelte-awesome-color-picker";

  const validFileTypes = [
    "image/png",
    "image/jpeg",
    "image/jpg",
    "image/gif",
    "image/webp",
  ];

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D | null;
  let imgSrc: string | null = $state(null);
  let cropPos = $state({ x: 0, y: 0 });
  let isDragging = false;
  let lastTouchY = 0;
  let image: HTMLImageElement | null = $state(null);
  let imgElement: HTMLImageElement | null = $state(null);
  let downloadURL: string | null = $state(null);
  let colorPicker = $state("#ffffff");
  let fileInput: HTMLInputElement;

  function onUpload(file: File) {
    cropPos = { x: 0, y: 0 };

    const img = new Image();
    image = img;
    imgSrc = URL.createObjectURL(file);
    img.src = imgSrc;
    img.onload = () => {
      const horizontal = (img.width / img.height) > (88 / 31);
      drawImage();
    }
  }

  function onDrop(e: DragEvent) {
    e.preventDefault();
    const files = e.dataTransfer?.files;
    if (files && files.length > 0) {
      if (validFileTypes.includes(files[0].type)) {
        onUpload(files[0]);
      } else {
        console.error("invalid file type");
      }
    } else {
      console.error("no file dropped");
    }
  }

  function fileInputChange(e: Event) {
    const target = e.target as HTMLInputElement;
    const files = target.files;
    if (files && files.length > 0) {
      if (validFileTypes.includes(files[0].type)) {
        onUpload(files[0]);
      } else {
        console.error("invalid file type");
      }
    }
  }

  function clamp(num: number, min: number, max: number) {
    return Math.min(Math.max(num, min), max);
  }

  function setDownloadURL() {
    canvas.toBlob((blob) => {
      if (blob) {
        downloadURL = URL.createObjectURL(blob);
      }
    }, "image/png");
  }

  function onCropMouseDown(e: MouseEvent) {
    e.preventDefault();
    isDragging = true;
  }

  function drawImage() {
    if (!ctx || !image) return;

    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.drawImage(image, 0, -cropPos.y / (canvas.clientWidth / 88), 88, image.height * 88 / image.width);
    ctx.strokeStyle = colorPicker;
    ctx.strokeRect(0.5, 0.5, 87, 30);

    setDownloadURL();
  }

  function onMouseMove(e: MouseEvent) {
    if (!isDragging || !imgElement) return;

    cropPos.y += e.movementY;
    cropPos.y = clamp(cropPos.y, 0, imgElement.clientHeight - canvas.clientHeight);
    drawImage();
  }

  function onTouchStart(e: TouchEvent) {
    e.preventDefault();
    isDragging = true;
    lastTouchY = e.touches[0].clientY;
  }

  function onTouchMove(e: TouchEvent) {
    if (!isDragging || !imgElement) return;

    const touch = e.touches[0];
    const movementY = touch.clientY - lastTouchY;
    lastTouchY = touch.clientY;

    cropPos.y += movementY;
    cropPos.y = clamp(cropPos.y, 0, imgElement.clientHeight - canvas.clientHeight);
    drawImage();
  }

  function onTouchEnd(e: TouchEvent) {
    isDragging = false;
  }

  function onMouseUp(e: MouseEvent) {
    e.preventDefault();
    isDragging = false;
  }

  onMount(() => {
    ctx = canvas.getContext("2d");

    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);

    document.addEventListener("touchmove", onTouchMove, { passive: false });
    document.addEventListener("touchend", onTouchEnd);
  });
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  ondrop={onDrop}
  ondragover={e => e.preventDefault()}
  class="h-full p-8 flex justify-center items-center"
>
  <div class="flex flex-col gap-6 items-center sm:items-start">
    <h1 class="font-bold">88x31 BUTTON MAKER</h1>

    <div class="sm:w-[36rem] w-[20rem] relative">
      {#if image}
        <img
          bind:this={imgElement}
          src={imgSrc}
          alt="img preview"
          class="w-full brightness-[20%]"
        >
      {:else}
        <button
          onclick={() => fileInput.click()}
          class="size-full aspect-square border-3 border-dashed border-neutral-800 flex items-center justify-center cursor-pointer hover:border-neutral-700 duration-100"
        >
          drop or select an image
        </button>
      {/if}

      <canvas
        onmousedown={onCropMouseDown}
        ontouchstart={onTouchStart}
        bind:this={canvas}
        width={88}
        height={31}
        style:left="{cropPos.x}px"
        style:top="{cropPos.y}px"
        class="w-full pixelated absolute active:cursor-grabbing bg-red-500 {image ? 'cursor-grab' : 'pointer-events-none hidden'}"
      ></canvas>
    </div>

    <div class="flex gap-4 justify-between items-center w-full">
      <ColorPicker
        bind:hex={colorPicker}
        onInput={() => drawImage()}
        label="border color"
        isDialog={true}
        --picker-height="200px"
        --picker-width="200px"
        --slider-width="25px"
        --picker-indicator-size="25px"
        --picker-z-index="10"
        --border-radius="0px"
        --focus-color="white"
        --cp-bg-color="var(--color-neutral-950)"
        --cp-text-color="white"
        --cp-input-color="var(--color-neutral-900)"
        --cp-button-hover-color="var(--color-neutral-700)"
      />

      <a
        download="button.png"
        href={downloadURL}
        class="px-4 py-2 flex items-center gap-2 font-bold {downloadURL ? 'bg-white text-black hover:scale-105 active:scale-100 duration-100' : 'bg-neutral-800 text-neutral-600 cursor-not-allowed'}"
      >
        <iconify-icon icon="mingcute:download-2-fill" class="text-lg"></iconify-icon>
        <span>download</span>
      </a>
    </div>
  </div>
</div>

<input
  onchange={fileInputChange}
  bind:this={fileInput}
  accept={validFileTypes.join(",")}
  type="file"
  name="image"
  id="image"
  class="hidden"
  multiple={false}
/>
