<template>
  <div class="flex gap-1 items-center">
    <template
      v-if="
        chapterStore.activeAudio &&
        chapterStore.activeTimestamp?.verse_key === verseKey &&
        !chapterStore.IsStoped
      "
    >
      <button @click="chapterStore.stopAudio" class="btn btn-sm btn-circle">
        <Icon name="iconamoon:player-stop-fill"></Icon>
      </button>
      <template v-if="chapterStore.isPaused">
        <button @click="chapterStore.playAudio" class="btn btn-sm btn-circle">
          <Icon name="iconamoon:player-play-fill"></Icon>
        </button>
      </template>
      <template v-if="chapterStore.isPlayed">
        <button
          @click="chapterStore.pauseAudio"
          class="btn btn-sm btn-circle btn-primary"
        >
          <Icon name="iconamoon:player-pause-fill"></Icon>
        </button>
      </template>
    </template>
    <template v-else>
      <button @click="initiateAudio()" class="btn btn-sm btn-circle">
        <Icon name="iconamoon:player-play-fill"></Icon>
      </button>
    </template>
  </div>
</template>

<script setup lang="ts">
import { useChapterStore } from "~/store/chapter";
const props = defineProps<{
  verseKey: string;
}>();

const { $api } = useNuxtApp();
const chapterStore = useChapterStore();

function fetchChapterRecitations(chapterNumber: number) {
  return $api.chapterRecitations.getByChapterNumber(chapterNumber);
}

async function getChapterRecitations() {
  if (chapterStore.activeChapter) {
    const chapterNumber = chapterStore.activeChapter.id;
    const chapterRecitations = await fetchChapterRecitations(chapterNumber);
    chapterStore.setActiveChapterRecitations(chapterRecitations.audio_file);
  }
}

async function initiateAudio() {
  if (chapterStore.activeChapter) {
    if (!chapterStore.activeChapterRecitations) await getChapterRecitations();
    if (!chapterStore.activeAudio) chapterStore.initiateAudio();

    chapterStore.initiateAudioEventListener();

    if (chapterStore.activeChapterRecitations) {
      const verseTimestamp = chapterStore.getVerseTimestamp(props.verseKey);
      if (verseTimestamp) {
        chapterStore.setAudioCurrentTime(verseTimestamp?.timestamp_from);
      }

      chapterStore.playAudio();
    }
  }
}
</script>
