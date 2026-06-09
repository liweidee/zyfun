<template>
  <div class="container-aside-wrap container-aside-manga">
    <div v-if="!active.profile" class="container-wrap main-wrap">
      <div class="info-wrap">
        <div class="new-title-wrap">
          <div class="new-title-name">
            <div class="title txthide txthide1">{{ infoConf.vod_name }}</div>
            <div class="title-unfold intro-unfold" @click="active.profile = true">
              <span>{{ $t('pages.player.film.desc') }}</span>
              <span class="icon-title-right"><chevron-right-s-icon /></span>
            </div>
          </div>
          <div class="new-title-feature txthide txthide1">
            <span class="meta-info heat">{{ infoConf.vod_score ? infoConf.vod_score : '0.0' }} </span>
            <span v-show="infoConf.type_name" class="meta-info">{{ infoConf.type_name }}</span>
            <span v-show="infoConf.vod_area" class="meta-info">{{ infoConf.vod_area }}</span>
            <span v-show="infoConf.vod_year" class="meta-info">{{ infoConf.vod_year }}</span>
          </div>
        </div>
        <div class="play-paction">
          <div class="paction-item like" @click="handleSwitchStar">
            <heart-filled-icon v-if="starData.id" class="icon" />
            <heart-icon v-else class="icon" />
            <span class="tip">{{ $t('pages.moment.star.title') }}</span>
          </div>
          <t-divider layout="vertical" />
          <div class="paction-item share" @click="MessagePlugin.info('暂不支持分享')">
            <share1-icon class="icon" />
            <span class="tip">{{ $t('component.share.title') }}</span>
          </div>
          <t-divider layout="vertical" />
          <div class="paction-item more">
            <t-dropdown trigger="click">
              <more-icon />
              <t-dropdown-menu>
                <t-dropdown-item>
                  <div class="setting-item" @click="handleSettingDialog">
                    <setting-icon />
                    {{ $t('pages.player.function.setting') }}
                  </div>
                </t-dropdown-item>
              </t-dropdown-menu>
            </t-dropdown>
          </div>
        </div>
        <dialog-download-view
          v-model:visible="active.download"
          :episode="downloadFormData.episode"
          :current="downloadFormData.current"
        />
        <dialog-setting-view
          v-model:visible="active.setting"
          type="film"
          :data="settingFormData"
          :time="processConf"
          @change="onSettingChange"
        />
      </div>

      <div class="anthology-container manga-anthology">
        <div class="anthology-series-wrap">
          <tag-nav
            class="anthology-series-nav"
            :list="navOptions"
            :active="active.nav"
            @change="handleSwitchSeriesTab"
          />
          <div class="anthology-series-extra">
            <div v-show="activeAnalyzeList.length" class="anthology-series-parse">
              <t-dropdown placement="bottom" :max-height="250">
                <t-button class="anthology-series-btn" theme="default" variant="text" auto-width>
                  {{ $t('pages.parse.title') }}
                  <template #suffix><chevron-down-icon /></template>
                </t-button>
                <t-dropdown-menu>
                  <t-dropdown-item
                    v-for="item in activeAnalyzeList"
                    :key="item.id"
                    :active="item.id === active.analyzeId"
                    @click="handleSwitchParse(item.id)"
                  >
                    <span>{{ item.name }}</span>
                  </t-dropdown-item>
                </t-dropdown-menu>
              </t-dropdown>
            </div>
            <div class="anthology-series-reverse">
              <t-button
                class="anthology-series-btn"
                theme="default"
                variant="text"
                auto-width
                @click="reverseOrderEvent"
              >
                <template #suffix>
                  <order-descending-icon v-if="active.reverseOrder" class="reverse-icon" />
                  <order-ascending-icon v-else class="reverse-icon" />
                </template>
              </t-button>
            </div>
          </div>
        </div>
        <div class="box-anthology-wrap">
          <div v-if="active.nav === 'episode'" class="box-anthology-item box-anthology-episode">
            <div class="box-anthology-wrap">
              <div class="box-anthology-item">
                <div v-if="lineList.length > 1" class="box-anthology-header">
                  <title-menu :list="lineList" :active="active.filmSource" class="nav" @change="handleSwitchLine" />
                </div>
                <div class="box-anthology-content">
                  <div class="grid-wrap">
                    <div
                      v-for="(item, chapIndex) in activeSessionList"
                      :key="chapIndex"
                      class="item-wrap"
                      :class="[`${item.text}$${item.link}` === active.filmIndex ? 'is-active' : '']"
                      @click="handleSwitchSeason(item, chapIndex)"
                    >
                      <div class="list-item">
                        <t-tooltip :content="item.text">
                          <div class="title txthide txthide1">{{ item.text }}</div>
                        </t-tooltip>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div v-if="active.nav === 'recommend'" class="box-anthology-item box-anthology-recommend">
            <div class="box-anthology-content">
              <t-list class="list-wrap" :scroll="{ rowHeight: 80, type: 'virtual' }">
                <t-list-item
                  v-for="(item, idx) in recommendList"
                  :key="idx"
                  class="item-wrap"
                  @click="handleSwitchRecommendItem(item)"
                >
                  <div class="list-item">
                    <div class="logo">
                      <t-image
                        class="logo-lazy"
                        fit="cover"
                        :src="item.vod_pic"
                        :lazy="true"
                        :loading="renderDefaultLazy"
                        :error="renderDefaultLazy"
                      />
                    </div>
                    <div class="title txthide txthide2">{{ item.vod_name }}</div>
                  </div>
                </t-list-item>
              </t-list>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-else class="container-wrap intro-wrap">
      <div class="side-head">
        <div class="header">{{ $t('pages.player.film.desc') }}</div>
        <div class="close-btn" @click="active.profile = false">
          <close-icon class="t-icon t-icon-close" />
        </div>
      </div>
      <div class="side-body">
        <div class="new-intro-base">
          <t-image
            class="new-intro-img"
            :src="infoConf.vod_pic"
            fit="cover"
            shape="round"
            :lazy="true"
            :loading="renderDefaultLazy"
            :error="renderDefaultLazy"
          />
          <h4 class="title txthide txthide2">{{ infoConf.vod_name }}</h4>
        </div>
        <div class="new-intro-detail">
          <div class="new-intro-case">
            <div class="new-intro-title txthide txthide1">{{ $t('pages.player.film.info.background') }}</div>
            <div class="new-intro-content">
              <span class="txt" v-html="infoConf.vod_content || $t('common.unknown')"></span>
            </div>
          </div>
          <div class="new-intro-case">
            <div class="new-intro-title txthide txthide1">{{ $t('pages.player.film.info.actors') }}</div>
            <div class="new-intro-content">
              <div class="new-intro-roles new-intro-director">
                <span class="intro-role-title">{{ $t('pages.player.film.info.director') }}: </span>
                <span class="intro-role-subtitle">
                  {{ infoConf.vod_director || $t('common.unknown') }}
                </span>
              </div>
              <div class="new-intro-roles new-intro-actor">
                <span class="intro-role-title">{{ $t('pages.player.film.info.actor') }}: </span>
                <span class="intro-role-subtitle">
                  {{ infoConf.vod_actor || $t('common.unknown') }}
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup lang="tsx">
import {
  isArray,
  isArrayEmpty,
  isNil,
  isObject,
  isObjectEmpty,
  isPositiveFiniteNumber,
} from '@shared/modules/validate';
import type { ICmsInfo, ICmsInfoEpisode, IRecMatch } from '@shared/types/cms';
import type { IModels } from '@shared/types/db';
import {
  ChevronDownIcon,
  ChevronRightSIcon,
  CloseIcon,
  HeartFilledIcon,
  HeartIcon,
  MoreIcon,
  OrderAscendingIcon,
  OrderDescendingIcon,
  SettingIcon,
  Share1Icon,
} from 'tdesign-icons-vue-next';
import { MessagePlugin } from 'tdesign-vue-next';
import type { PropType } from 'vue';
import { computed, onMounted, ref, toRaw, watch } from 'vue';

import { fetchCmsDetail, fetchCmsPlay, fetchCmsSearch, fetchRecMatch } from '@/api/film';
import { fetchAnalyzeActive } from '@/api/parse';
import LazyBg from '@/components/lazy-bg/index.vue';
import TagNav from '@/components/tag-nav/index.vue';
import TitleMenu from '@/components/title-menu/index.vue';
import { emitterChannel } from '@/config/emitterChannel';
import type { IStorePlayer } from '@/config/player';
import { useHistory } from '@/hooks/useHistory';
import { useStar } from '@/hooks/useStar';
import { t } from '@/locales';
import type { IVideoOptions, IVideoProcess } from '@/types/player';
import emitter from '@/utils/emitter';

import DialogDownloadView from './DialogDownload.vue';
import DialogSettingView from './DialogSetting.vue';

const props = defineProps({
  store: {
    type: Object as PropType<IStorePlayer>,
    default: () => ({}),
  },
  process: {
    type: Object as PropType<IVideoProcess>,
    default: () => ({ currentTime: 0, duration: 0 }),
  },
});

const emits = defineEmits(['update', 'barrage', 'create', 'pause', 'seek', 'manga-content-change']);

const renderDefaultLazy = () => <LazyBg class="render-icon" />;

const DEFAULT_SKIP_TIME = 90;

const infoConf = ref(props.store.data.info as ICmsInfo);
const extraConf = ref(props.store.data.extra);
const playerConf = ref(props.store.setting);
const processConf = ref(props.process);

const lineList = ref<{ type_id: string; type_name: string }[]>([]);
const recommendList = ref<IRecMatch[]>([]);
const analyzeConfig = ref({
  default: {} as IModels['analyze'],
  list: [] as IModels['analyze'][],
});

const videoData = ref<IVideoOptions>({
  url: '',
  playEnd: false,
  watchTime: 0,
  duration: 0,
  skipTimeInStart: DEFAULT_SKIP_TIME,
  skipTimeInEnd: DEFAULT_SKIP_TIME,
});

const downloadFormData = ref({
  episode: {} as ICmsInfo['vod_episode'],
  current: '',
});
const settingFormData = ref({
  skipHeadAndEnd: false,
  skipTimeInStart: DEFAULT_SKIP_TIME,
  skipTimeInEnd: DEFAULT_SKIP_TIME,
  playNextPreload: false,
  playNextEnabled: true,
  skipAd: false,
});

const active = ref({
  watch: true,
  profile: false,
  nav: 'episode',
  reverseOrder: true,
  share: false,
  download: false,
  setting: false,
  analyzeId: '',
  filmIndex: '',
  filmSource: '',
  transitioning: false,
});

const navOptions = computed(() => [
  { value: 'episode', label: t('pages.player.film.anthology') },
  ...(recommendList.value.length ? [{ value: 'recommend', label: t('pages.player.film.recommend') }] : []),
]);

const activeAnalyzeList = computed(() => {
  const flag = active.value.filmSource;
  return analyzeConfig.value.list.filter((item) => (item.flag || []).includes(flag));
});

const activeSessionList = computed(() => {
  const flag = active.value.filmSource;
  return infoConf.value.vod_episode?.[flag] || [];
});

// 收藏相关
const { starData, getStarData, handleSwitchStar } = useStar({
  source: infoConf,
  getQuery: (info) => ({
    relateId: extraConf.value.active.key,
    videoId: info.vod_id,
    type: 4,
  }),
  createDoc: (info) => ({
    type: 4,
    relateId: extraConf.value.active.key,
    videoId: info.vod_id,
    videoImage: info.vod_pic,
    videoName: info.vod_name,
    videoType: '漫画',
    videoRemarks: info.vod_remarks || '',
  }),
});

// 历史记录
const { getHistoryData, throttleSaveHistory } = useHistory({
  source: infoConf,
  getQuery: (info) => ({
    relateId: extraConf.value.active.key,
    videoId: info.vod_id,
    type: 8,
  }),
  createDoc: (info) => ({
    type: 8,
    relateId: extraConf.value.active.key,
    siteSource: active.value.filmSource,
    playEnd: videoData.value.playEnd,
    videoId: info.vod_id,
    videoImage: info.vod_pic,
    videoName: info.vod_name,
    videoIndex: active.value.filmIndex,
    watchTime: isPositiveFiniteNumber(videoData.value.watchTime) ? videoData.value.watchTime : 0,
    duration: isPositiveFiniteNumber(videoData.value.duration) ? videoData.value.duration : 0,
    skipTimeInStart: videoData.value.skipTimeInStart,
    skipTimeInEnd: videoData.value.skipTimeInEnd,
  }),
  onLoaded: (history) => {
    const { skipHeadAndEnd } = playerConf.value;
    const skipTimeInStart = history.skipTimeInStart ?? DEFAULT_SKIP_TIME;
    const skipTimeInEnd = history.skipTimeInEnd ?? DEFAULT_SKIP_TIME;
    const duration = history.duration ?? 0;
    const rawWatchTime = history.watchTime ?? 0;
    const playEnd = history.playEnd ?? false;
    videoData.value = {
      ...videoData.value,
      skipTimeInStart,
      skipTimeInEnd,
      duration,
      watchTime: skipHeadAndEnd ? Math.max(rawWatchTime, skipTimeInStart) : rawWatchTime,
      playEnd,
    };
  },
});

// 解析漫画播放结果，提取图片列表
const parseMangaPlayResult = (url: string): string[] => {
  if (url.startsWith('pics://')) {
    const imgList = url.substring(7).split('&&');
    return imgList.filter((img) => img && img.trim());
  }
  return [];
};

watch(
  () => props.store,
  (val) => {
    infoConf.value = val.data.info as ICmsInfo;
    extraConf.value = val.data.extra;
    playerConf.value = val.setting;
  },
  { deep: true },
);
watch(
  () => props.process,
  (val) => {
    if (!active.value.transitioning) processConf.value = val;
  },
);

onMounted(() => setup());

const handleSwitchSeriesTab = (val: string) => {
  active.value.nav = val;
};

// const handleDownloadDialog = () => {
//   downloadFormData.value = {
//     episode: infoConf.value.vod_episode,
//     current: videoData.value.url,
//   };
//   active.value.download = true;
// };

// 分享：复制当前章节链接
// const handleCopyShareLink = async () => {
//   try {
//     let shareUrl = videoData.value.url;
//     if (!shareUrl && active.value.filmIndex) {
//       const [, link] = active.value.filmIndex.split('$');
//       shareUrl = link;
//     }

//     if (!shareUrl) {
//       MessagePlugin.warning('暂无分享链接');
//       return;
//     }

//     await navigator.clipboard.writeText(shareUrl);
//     MessagePlugin.success('链接已复制到剪贴板');
//   } catch (error) {
//     console.error('[Share] 复制失败:', error);
//     MessagePlugin.error('复制失败，请手动复制');
//   }
// };

const handleSettingDialog = () => {
  settingFormData.value = {
    skipHeadAndEnd: playerConf.value.skipHeadAndEnd,
    playNextPreload: playerConf.value.playNextPreload,
    playNextEnabled: playerConf.value.playNextEnabled,
    skipAd: playerConf.value.skipAd,
    skipTimeInStart: videoData.value.skipTimeInStart,
    skipTimeInEnd: videoData.value.skipTimeInEnd,
  };
  active.value.setting = true;
};

const onSettingChange = (item: typeof settingFormData.value) => {
  const {
    skipTimeInStart = DEFAULT_SKIP_TIME,
    skipTimeInEnd = DEFAULT_SKIP_TIME,
    skipHeadAndEnd,
    playNextPreload,
    playNextEnabled,
    skipAd,
  } = item;

  videoData.value.skipTimeInStart = skipTimeInStart;
  videoData.value.skipTimeInEnd = skipTimeInEnd;
  playerConf.value.skipHeadAndEnd = skipHeadAndEnd;
  playerConf.value.playNextPreload = playNextPreload;
  playerConf.value.playNextEnabled = playNextEnabled;
  playerConf.value.skipAd = skipAd;

  emits('update', { setting: playerConf.value });
};

const handleSwitchLine = (id: string) => {
  active.value.filmSource = id;
};

const reverseOrderEvent = () => {
  const episodes = infoConf.value.vod_episode;
  if (!episodes) return;

  infoConf.value.vod_episode = Object.fromEntries(
    Object.entries(episodes).map(([key, arr]) => [key, arr.toReversed()]),
  );
  active.value.reverseOrder = !active.value.reverseOrder;
};

const handleSwitchSeason = async (item: ICmsInfoEpisode, _index: number = -1) => {
  const currentLine = active.value.filmSource;

  active.value.filmIndex = `${item.text}$${item.link}`;
  active.value.watch = false;

  try {
    const playRes = await fetchCmsPlay({
      uuid: extraConf.value.active.id,
      play: item.link,
      flag: currentLine,
    });

    if (!playRes.url) throw new Error('No Play URL');

    const images = parseMangaPlayResult(playRes.url);
    if (images.length === 0) {
      throw new Error('未获取到图片列表');
    }

    // 发送内容到父组件显示
    emits('manga-content-change', {
      images,
      currentIndex: 0,
      title: item.text,
      chapterLink: item.link,
    });

    // 保存历史记录
    throttleSaveHistory();

    // 记录历史
    videoData.value.url = playRes.url;
    videoData.value.watchTime = 0;
    videoData.value.duration = images.length;
    videoData.value.playEnd = false;
  } catch (error) {
    console.error('[Manga] 加载失败:', error);
    MessagePlugin.error('加载漫画失败，请稍后重试');
  }
};

const handleSwitchParse = async (id: string) => {
  active.value.analyzeId = id;
  if (active.value.filmIndex) {
    const [text, link] = active.value.filmIndex.split('$');
    await handleSwitchSeason({ text, link });
  }
};

const getAnalyzeConfig = async () => {
  try {
    const resp = await fetchAnalyzeActive();
    if (resp?.default?.id) {
      analyzeConfig.value.default = resp.default;
      active.value.analyzeId = resp.default.id;
    }
    if (resp?.list) analyzeConfig.value.list = resp.list;
  } catch (error) {
    console.error('[player][getAnalyzeConfig]', error);
  }
};

const fetchRecommend = async () => {
  try {
    const { vod_name: name } = infoConf.value;
    const year = infoConf.value.vod_year || new Date().getFullYear();
    const res = await fetchRecMatch({ name, year });
    recommendList.value = res || [];
  } catch {
    recommendList.value = [];
  }
};

const handleSwitchRecommendItem = async (item: IRecMatch) => {
  const site = extraConf.value.active;

  const searchResp = await fetchCmsSearch({
    uuid: site.id,
    wd: item.vod_name,
    page: 1,
  });

  if (!isArray(searchResp.list) || isArrayEmpty(searchResp.list) || isNil(searchResp.list[0]?.vod_id)) {
    MessagePlugin.warning(t('pages.player.message.noRecMatch'));
    return;
  }

  const detailResp = await fetchCmsDetail({
    uuid: site.id,
    ids: searchResp.list[0].vod_id,
  });

  if (
    !isArray(detailResp.list) ||
    isArrayEmpty(detailResp.list) ||
    !isObject(detailResp.list[0]?.vod_episode) ||
    isObjectEmpty(detailResp.list[0]?.vod_episode)
  ) {
    MessagePlugin.warning(t('pages.film.message.noDetailInfo'));
    return;
  }

  const detail = detailResp.list[0];
  const searchItem = searchResp.list[0];

  const info = {
    ...detail,
    ...(detail?.vod_id ? {} : { vod_id: searchItem?.vod_id }),
    ...(detail?.vod_name ? {} : { vod_name: searchItem?.vod_name }),
    ...(detail?.vod_pic ? {} : { vod_pic: searchItem?.vod_pic }),
  };

  recommendList.value = [];
  active.value.reverseOrder = true;
  active.value.nav = 'episode';

  videoData.value = {
    url: '',
    playEnd: false,
    watchTime: 0,
    duration: 0,
    skipTimeInStart: DEFAULT_SKIP_TIME,
    skipTimeInEnd: DEFAULT_SKIP_TIME,
  };

  await emits('update', {
    data: toRaw({
      info,
      extra: extraConf.value,
    }),
  });

  setup();
};

const setup = async () => {
  const episode = infoConf.value.vod_episode;
  if (!isObject(episode) || isObjectEmpty(episode)) return;

  const episodeKeys = Object.keys(episode);
  const filmSource = episodeKeys[0];
  const flimEpisode = episode[filmSource]?.[0];

  if (!isObject(flimEpisode) || isObjectEmpty(flimEpisode)) return;
  const filmIndex = `${flimEpisode.text}$${flimEpisode.link}`;

  lineList.value = episodeKeys.map((key) => ({ type_id: key, type_name: key }));

  active.value.filmSource = filmSource;
  active.value.filmIndex = filmIndex;

  await getAnalyzeConfig();

  // 获取历史记录
  await getHistoryData();

  getStarData();
  fetchRecommend();

  // 自动加载第一集
  if (flimEpisode && flimEpisode.link) {
    await handleSwitchSeason(flimEpisode, 0);
  }
};

emitter.on(emitterChannel.COMP_MULTI_PLAYER_PLAYNEXT, async () => {});
</script>
<style lang="less" scoped>
.container-aside-manga {
  display: flex;
  flex-direction: column;
  height: 100%;

  .container-wrap {
    flex-shrink: 0;
  }
}
</style>
