<template>
  <div class="layout-player">
    <t-layout :class="[`${prefix}-layout`]">
      <t-header class="drag-region" :class="[`${prefix}-header`, active.headerPin ? 'pin' : '']">
        <l-header :title="title" :browse="headerFormData.browse" @browse="handleBrowse" />
      </t-header>
      <t-layout :class="[`${prefix}-main`]">
        <t-content :class="[`${prefix}-content`]">
          <div :class="[`${prefix}-content-container`]" style="position: relative">
            <!-- 视频播放器 - 始终存在，但对于漫画/小说隐藏 -->
            <multi-player
              ref="playerRef"
              class="media-player"
              :class="{ 'player-hidden': storePlayer.type === 'manga' || storePlayer.type === 'novel' }"
              @update-time="onTimeUpdate"
            />

            <!-- 漫画阅读器 - 叠加层 -->
            <div v-if="storePlayer.type === 'manga'" class="content-overlay manga-overlay">
              <div v-if="mangaImages.length" class="manga-viewer">
                <div class="manga-image-wrapper">
                  <img :key="mangaCurrentIndex" :src="mangaImages[mangaCurrentIndex]" class="manga-image" />
                </div>
                <div class="manga-controls">
                  <t-button
                    theme="default"
                    variant="outline"
                    :disabled="mangaCurrentIndex <= 0"
                    @click="prevMangaImage"
                  >
                    <chevron-left-icon class="button-icon" /> {{ t('pages.player.manga.prevPage') }}
                  </t-button>
                  <span class="page-info">{{ mangaCurrentIndex + 1 }} / {{ mangaImages.length }}</span>
                  <t-button
                    theme="default"
                    variant="outline"
                    :disabled="mangaCurrentIndex >= mangaImages.length - 1"
                    @click="nextMangaImage"
                  >
                    {{ t('pages.player.manga.nextPage') }} <chevron-right-icon class="button-icon" />
                  </t-button>
                </div>
              </div>
              <div v-else class="empty-viewer">
                <t-empty :description="t('pages.player.manga.emptyDesc')" />
              </div>
            </div>

            <!-- 小说阅读器 - 叠加层 -->
            <div v-else-if="storePlayer.type === 'novel'" class="content-overlay novel-overlay">
              <div v-if="novelContent" class="novel-viewer">
                <div class="novel-header">
                  <h3 class="novel-title">{{ novelTitle }}</h3>
                </div>
                <div class="novel-content-wrapper">
                  <div
                    class="novel-content"
                    :style="{ fontSize: `${novelFontSize}px` }"
                    v-html="novelFormattedContent"
                  ></div>
                </div>
                <div class="novel-controls">
                  <t-button
                    theme="default"
                    variant="outline"
                    :disabled="novelChapterIndex <= 0"
                    @click="prevNovelChapter"
                  >
                    <chevron-left-icon class="button-icon" /> {{ t('pages.player.novel.prevChapter') }}
                  </t-button>
                  <t-select
                    v-model="novelChapterIndex"
                    :options="novelChapterOptions"
                    style="width: 200px"
                    @change="onNovelChapterChange"
                  />
                  <t-button
                    theme="default"
                    variant="outline"
                    :disabled="novelChapterIndex >= novelChapterList.length - 1"
                    @click="nextNovelChapter"
                  >
                    {{ t('pages.player.novel.nextChapter') }} <chevron-right-icon class="button-icon" />
                  </t-button>
                </div>
                <div class="novel-font-controls">
                  <t-button theme="default" size="small" variant="outline" @click="decreaseNovelFontSize">
                    {{ t('pages.player.novel.fontSizeDecrease') }}
                  </t-button>
                  <span class="font-size-indicator">{{ novelFontSize }}px</span>
                  <t-button theme="default" size="small" variant="outline" @click="increaseNovelFontSize">
                    {{ t('pages.player.novel.fontSizeIncrease') }}
                  </t-button>
                </div>
              </div>
              <div v-else class="empty-viewer">
                <t-empty :description="t('pages.player.novel.emptyDesc')" />
              </div>
            </div>

            <div class="dock-show" @click="toggleAside">
              <chevron-left-icon v-if="active.aside" class="dock-icon" />
              <chevron-right-icon v-else class="dock-icon" />
            </div>
          </div>
        </t-content>
        <t-aside v-show="!active.aside" :class="[`${prefix}-aside`]">
          <div :class="[`${prefix}-aside-container`]">
            <component
              :is="currentAsideComponent"
              ref="asideComponentRef"
              class="container-aside"
              :store="playerStoreFormData"
              :process="processFormData"
              @barrage="updateBarrage"
              @create="handlePlayerCreate"
              @pause="handlePlayerPause"
              @seek="handlePlayerSeek"
              @update="updateConf"
              @manga-content-change="onMangaContentChange"
              @novel-content-change="onNovelContentChange"
            />
          </div>
        </t-aside>
      </t-layout>
    </t-layout>
  </div>
</template>
<script setup lang="ts">
const asideComponentRef = ref<any>(null);
import { APP_NAME } from '@shared/config/appinfo';
import { SYSTEM_M3U8_AD_REMOVE_API } from '@shared/config/env';
import { IPC_CHANNEL } from '@shared/config/ipcChannel';
import { WINDOW_NAME } from '@shared/config/window';
import { isArray, isArrayEmpty } from '@shared/modules/validate';
import type { IBarrageResult } from '@shared/types/barrage';
import { merge } from 'es-toolkit';
import { ChevronLeftIcon, ChevronRightIcon } from 'tdesign-icons-vue-next';
import { computed, defineAsyncComponent, onMounted, onUnmounted, ref, shallowRef, useTemplateRef, watch } from 'vue';

import type { IMultiPlayerOptions, MultiPlayerInstance } from '@/components/multi-player';
import { MultiPlayer } from '@/components/multi-player';
import { prefix } from '@/config/global';
import type { IStorePlayer } from '@/config/player';
import { t } from '@/locales';
import { usePlayerStore } from '@/store';
import type { IVideoProcess } from '@/types/player';

import LHeader from './components/Header.vue';

const storePlayer = usePlayerStore();

const componentMap = {
  film: defineAsyncComponent(() => import('./components/AsideFilm.vue')),
  live: defineAsyncComponent(() => import('./components/AsideLive.vue')),
  parse: defineAsyncComponent(() => import('./components/AsideParse.vue')),
  manga: defineAsyncComponent(() => import('./components/AsideManga.vue')),
  novel: defineAsyncComponent(() => import('./components/AsideNovel.vue')),
};

const playerRef = useTemplateRef<MultiPlayerInstance | null>('playerRef');
const currentAsideComponent = shallowRef(componentMap[storePlayer.type]);

const playerFormData = ref<IMultiPlayerOptions>({
  url: '',
  autoplay: true,
  next: false,
  quality: [],
  isLive: false,
  headers: {},
  type: 'auto',
  startTime: 0,
  container: 'play-mse',
});
const processFormData = ref<IVideoProcess>({
  currentTime: 0,
  duration: 0,
});
const headerFormData = ref({
  browse: false,
});
const playerStoreFormData = computed<IStorePlayer>(() => storePlayer.$state);

const active = ref({
  aside: false,
  headerPin: false,
});

const title = computed(() => {
  const type = storePlayer.type;
  const info = storePlayer.data.info;

  if (type === 'film') return info.vod_name || APP_NAME;
  if (type === 'manga') return info.vod_name || APP_NAME;
  if (type === 'novel') return info.vod_name || APP_NAME;
  return info.name || APP_NAME;
});

// 漫画相关状态
const mangaImages = ref<string[]>([]);
const mangaCurrentIndex = ref(0);

// 小说相关状态
const novelTitle = ref('');
const novelContent = ref('');
const novelFontSize = ref(18);
const novelChapterIndex = ref(0);
const novelChapterList = ref<{ text: string; link: string }[]>([]);

const novelFormattedContent = computed(() => {
  const content = novelContent.value;
  if (!content) return '';
  const paragraphs = content.split(/\n/);
  return paragraphs
    .filter((p) => p && p.trim())
    .map((p) => `<p style="text-indent: 2em; margin-bottom: 1em;">${p}</p>`)
    .join('');
});

const novelChapterOptions = computed(() => {
  return novelChapterList.value.map((ch, idx) => ({
    label: ch.text,
    value: idx,
  }));
});

// 漫画控制方法
const prevMangaImage = () => {
  if (mangaCurrentIndex.value > 0) {
    mangaCurrentIndex.value--;
  }
};

const nextMangaImage = () => {
  if (mangaCurrentIndex.value < mangaImages.value.length - 1) {
    mangaCurrentIndex.value++;
  }
};

// 小说控制方法
const prevNovelChapter = async () => {
  if (novelChapterIndex.value > 0) {
    const prev = novelChapterList.value[novelChapterIndex.value - 1];
    if (prev && asideComponentRef.value?.loadChapter) {
      await asideComponentRef.value.loadChapter(prev, novelChapterIndex.value - 1);
    }
  }
};

const nextNovelChapter = async () => {
  if (novelChapterIndex.value < novelChapterList.value.length - 1) {
    const next = novelChapterList.value[novelChapterIndex.value + 1];
    if (next && asideComponentRef.value?.loadChapter) {
      await asideComponentRef.value.loadChapter(next, novelChapterIndex.value + 1);
    }
  }
};

const onNovelChapterChange = async (value: number) => {
  if (value >= 0 && value < novelChapterList.value.length) {
    const chapter = novelChapterList.value[value];
    if (chapter && asideComponentRef.value?.loadChapter) {
      await asideComponentRef.value.loadChapter(chapter, value);
    }
  }
};

const increaseNovelFontSize = () => {
  if (novelFontSize.value < 32) {
    novelFontSize.value += 2;
  }
};

const decreaseNovelFontSize = () => {
  if (novelFontSize.value > 12) {
    novelFontSize.value -= 2;
  }
};

// 接收子组件内容事件
const onMangaContentChange = (data: { images: string[]; currentIndex: number; title: string; chapterLink: string }) => {
  mangaImages.value = data.images;
  mangaCurrentIndex.value = data.currentIndex;
};

const onNovelContentChange = (data: {
  title: string;
  content: string;
  chapterLink: string;
  chapterIndex: number;
  totalChapters: number;
  chapterList: { text: string; link: string }[];
}) => {
  novelTitle.value = data.title;
  novelContent.value = data.content;
  novelChapterIndex.value = data.chapterIndex;
  novelChapterList.value = data.chapterList;
};

watch(
  () => storePlayer.setting.skipAd,
  (val) => {
    const watchTime = processFormData.value.currentTime;
    handlePlayerCreate({ ...playerFormData.value, skipAd: val, startTime: watchTime }, 'new');
  },
);

onMounted(() => setup());
onUnmounted(() => dispose());

const setup = () => {
  window.electron.ipcRenderer.on(IPC_CHANNEL.WINDOW_DESTROY, () => {
    storePlayer.updateConfig({ status: false });
    window.electron.ipcRenderer.send(IPC_CHANNEL.WINDOW_DESTROY_RELAY);
  });

  window.electron.ipcRenderer.on(IPC_CHANNEL.MEDIA_PAUSE, (_event, status) => {
    status === true ? playerRef.value?.pause() : playerRef.value?.play();
  });

  window.electron.ipcRenderer.on(IPC_CHANNEL.MEDIA_BROWSE, (_event, status) => {
    if (headerFormData.value.browse) {
      if (status) {
        window.electron.ipcRenderer.invoke(IPC_CHANNEL.WINDOW_HIDE, WINDOW_NAME.PLAYER);
        playerRef.value?.pause();
      } else {
        window.electron.ipcRenderer.invoke(IPC_CHANNEL.WINDOW_SHOW, WINDOW_NAME.PLAYER);
        playerRef.value?.play();
      }
    }
  });

  document.title = `${APP_NAME}(${t('pages.player.title')})`;
};

const dispose = () => {
  storePlayer.updateConfig({ status: false });
};

const toggleAside = () => {
  active.value.aside = !active.value.aside;
};

const handleBrowse = (val: boolean) => {
  headerFormData.value.browse = val;
};

const handleUrlAdRemove = (url: string, remove: boolean = false): string => {
  if (!url.startsWith('http')) return url;

  if (remove && !url.startsWith(SYSTEM_M3U8_AD_REMOVE_API)) {
    return `${SYSTEM_M3U8_AD_REMOVE_API}?url=${encodeURIComponent(url)}`;
  }
  if (!remove && url.startsWith(SYSTEM_M3U8_AD_REMOVE_API)) {
    return decodeURIComponent(url.replace(`${SYSTEM_M3U8_AD_REMOVE_API}?url=`, ''));
  }

  return url;
};

const onTimeUpdate = (time: IVideoProcess) => (processFormData.value = time);

const updateConf = (item: IStorePlayer) => storePlayer.updateConfig(item);

const updateBarrage = (item: IBarrageResult) => {
  setTimeout(() => {
    playerRef.value?.barrage(item.list, item.id);
  }, 0);
};

const handlePlayerCreate = async (
  item: IMultiPlayerOptions & { skipAd?: boolean },
  mode: 'switch' | 'new' = 'switch',
) => {
  const player = storePlayer.player;

  if (player.type === 'custom') {
    window.electron.ipcRenderer.invoke(IPC_CHANNEL.CALL_PLAYER, player.external, item.url);
  } else {
    const finalItem: IMultiPlayerOptions = {
      ...item,
      url: handleUrlAdRemove(item.url, item.skipAd),
      quality:
        isArray(item.quality) && !isArrayEmpty(item.quality)
          ? item.quality.map((q) => ({ name: q.name, url: handleUrlAdRemove(q.url, item.skipAd) }))
          : [],
    };
    playerFormData.value = merge(playerFormData.value, finalItem);
    await playerRef.value?.create(playerFormData.value, player.type, mode);
  }
};

const handlePlayerPause = () => playerRef.value?.pause();

const handlePlayerSeek = (time: number) => playerRef.value?.seek(time);
</script>
<style lang="less" scoped>
@import '@/style/player.less';

.media-player.player-hidden {
  visibility: hidden;
  opacity: 0;
  pointer-events: none;
}

.content-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: var(--td-bg-color-container);
  z-index: 10;
  overflow: auto;
}

.manga-overlay,
.novel-overlay {
  padding: var(--td-comp-paddingTB-m) var(--td-comp-paddingLR-m);
}

.manga-viewer {
  height: 100%;
  display: flex;
  flex-direction: column;

  .manga-image-wrapper {
    flex: 1;
    min-height: 0;
    overflow: auto;
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: #1a1a1a;
    border-radius: var(--td-radius-medium);

    .manga-image {
      max-width: 100%;
      max-height: 100%;
      object-fit: contain;
    }
  }

  .manga-controls {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: var(--td-size-4);
    padding: var(--td-comp-paddingTB-s) 0;

    :deep(.t-button) .t-button__text {
      display: inline-flex;
      align-items: center;
      gap: 4px;
    }
  }
}

.novel-viewer {
  height: 100%;
  display: flex;
  flex-direction: column;
  gap: var(--td-size-4);

  .novel-header {
    text-align: center;
    padding-bottom: var(--td-comp-paddingTB-m);
    border-bottom: 1px solid var(--td-border-level-1-color);

    .novel-title {
      margin: 0;
      font-size: var(--td-font-size-title-large);
      font-weight: 600;
      color: var(--td-text-color-primary);
    }
  }

  .novel-content-wrapper {
    flex: 1;
    min-height: 0;
    overflow-y: auto;
    padding: var(--td-comp-paddingTB-l) var(--td-comp-paddingLR-l);
    background-color: var(--td-bg-color-page);
    border-radius: var(--td-radius-medium);

    .novel-content {
      line-height: 1.8;
      color: var(--td-text-color-primary);

      :deep(p) {
        margin-bottom: var(--td-comp-margin-s);
      }
    }
  }

  .novel-controls {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: var(--td-size-4);
    padding: var(--td-comp-paddingTB-s) 0;

    :deep(.t-button) .t-button__text {
      display: inline-flex;
      align-items: center;
      gap: 4px;
    }
  }

  .novel-font-controls {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: var(--td-size-4);

    .font-size-indicator {
      min-width: 50px;
      text-align: center;
      font-size: var(--td-font-size-body-medium);
      color: var(--td-text-color-secondary);
    }
  }
}

.empty-viewer {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
