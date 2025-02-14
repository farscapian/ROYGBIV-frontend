<script lang="ts">
  import Button from './Button.svelte'
  import { prisms$ } from '../streams'
  import Slide from './Slide.svelte'
  import PrismSummary from './PrismSummary.svelte'
  import { goto } from '$app/navigation'
  import { fade } from 'svelte/transition'
  import plus from '../icons/plus'
  import arrowLeft from '../icons/arrow-left'
  import arrowRight from '../icons/arrow-right'

  type Slides = typeof slides
  type SlideStep = Slides[number]
  type SlideDirection = 'right' | 'left'

  let slides = [0] // Default slides
  let slide: SlideStep = 0
  let previousSlide: SlideStep = 0

  function back() {
    previousSlide = slides[slides.indexOf(slide) - 2]
    slide = slides[slides.indexOf(slide) - 1]
  }

  function next(to = slides[slides.indexOf(slide) + 1]) {
    previousSlide = slide
    slide = to
  }

  $: slideDirection = (
    slides.indexOf(previousSlide) > slides.indexOf(slide) ? 'right' : 'left'
  ) as SlideDirection

  // Update slides based on prism count
  $: {
    const prismSlides = Array.from({ length: $prisms$?.data.length - 1 }, (_, index) => index + 1)
    slides = [0, ...prismSlides]
  }
</script>

{#if !$prisms$.loading}
  {#if $prisms$.data.length}
    <div class="w-full max-w-sm">
      <div class="flex justify-between">
        <h1 in:fade|local={{ duration: 250 }} class="text-4xl">Your prisms</h1>
        <Button on:click={() => goto('/create')} format={'secondary'}
          ><div class="w-6">{@html plus}</div></Button
        >
      </div>

      {#each $prisms$.data as prism, i}
        <div in:fade|local={{ duration: 250 }}>
          {#if slide === i}
            <Slide direction={slideDirection}>
              <PrismSummary {prism} />
              <div class="mt-8 flex w-full justify-between">
                <Button disabled={i === 0} format="secondary" on:click={() => back()}
                  ><div class="w-6">{@html arrowLeft}</div></Button
                >
                <Button
                  disabled={slide === slides.length - 1}
                  format="secondary"
                  on:click={() => next()}><div class="w-6">{@html arrowRight}</div></Button
                >
              </div>
            </Slide>
          {/if}
        </div>
      {/each}
    </div>
  {:else}
    <div class="text-center">
      <p class="mb-6">Create your first prism!</p>
      <Button on:click={() => goto('/create')} format={'primary'}>Create</Button>
    </div>
  {/if}
{/if}
