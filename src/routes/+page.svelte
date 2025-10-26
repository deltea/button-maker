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
  let image: HTMLImageElement | null = $state(null);
  let imgElement: HTMLImageElement | null = $state(null);
  let downloadURL: string | null = $state(null);
  let colorPicker = $state("#ffffff");

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

  function onMouseUp(e: MouseEvent) {
    e.preventDefault();
    isDragging = false;
  }

  onMount(() => {
    ctx = canvas.getContext("2d");

    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);
  });
</script>

<!-- svelte-ignore a11y_no_static_element_interactions -->
<div
  ondrop={onDrop}
  ondragover={e => e.preventDefault()}
  class="h-full p-8 flex justify-center items-center"
>
  <div class="flex flex-col gap-6 itemscenter">
    <h1 class="font-bold">88x31 BUTTON MAKER</h1>

    <div class="w-[32rem] relative">
      {#if image}
        <img bind:this={imgElement} src={imgSrc} alt="img preview" class="w-full brightness-[20%]">
      {:else}
        <div class="size-full aspect-square border-4 border-dashed border-neutral-800 flex items-center justify-center">
          drop an image
        </div>
      {/if}
      <canvas
        onmousedown={onCropMouseDown}
        bind:this={canvas}
        width={88}
        height={31}
        style:left="{cropPos.x}px"
        style:top="{cropPos.y}px"
        class="w-full pixelated absolute"
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
        --focus-color="white"
        --cp-bg-color="var(--color-neutral-950)"
        --cp-text-color="white"
        --cp-input-color="var(--color-neutral-900)"
        --cp-button-hover-color="var(--color-neutral-700)"
      />

      <a
        download
        href={downloadURL}
        class="px-4 py-2 flex items-center gap-2 font-bold {downloadURL ? 'bg-white text-black hover:scale-105 active:scale-100 duration-100' : 'bg-neutral-800 text-neutral-600 cursor-not-allowed'}"
      >
        <iconify-icon icon="mingcute:download-2-fill" class="text-lg"></iconify-icon>
        <span>download</span>
      </a>
    </div>
  </div>

</div>
