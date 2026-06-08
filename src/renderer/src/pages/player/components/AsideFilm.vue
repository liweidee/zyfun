<template>
  <div class="container-aside-wrap container-aside-film">
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

          <div class="paction-item download" @click="handleDownloadDialog">
            <download-icon class="icon" />
            <span class="tip">{{ $t('pages.player.function.download') }}</span>
          </div>
          <t-divider layout="vertical" />

          <div class="paction-item share" @click="handleSharePopup">
            <t-popup trigger="click">
              <share1-icon class="icon" />
              <span class="tip">{{ $t('component.share.title') }}</span>

              <template #content>
                <share-popup :data="shareFormData" type="popup" />
              </template>
            </t-popup>
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

      <div class="anthology-container film-anthology">
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
                      v-for="(item, index) in activeSessionList"
                      :key="index"
                      class="item-wrap"
                      :class="[`${item.text}$${item.link}` === active.filmIndex ? 'is-active' : '']"
                      @click="handleSwitchSeason(item, index)"
                    >
                      <div class="list-item">
                        <t-tooltip :content="item.text">
                          <div class="title txthide txthide1">{{ reverseOrderIndex(index) }}</div>
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
                  v-for="(item, index) in recommendList"
                  :key="index"
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
import { PROXY_API } from '@shared/config/env';
import {
  isArray,
  isArrayEmpty,
  isBoolean,
  isHttp,
  isNil,
  isObject,
  isObjectEmpty,
  isPositiveFiniteNumber,
} from '@shared/modules/validate';
import type { ICmsInfo, ICmsInfoEpisode, IRecMatch } from '@shared/types/cms';
import type { IModels } from '@shared/types/db';
import { throttle } from 'es-toolkit';
import {
  ChevronDownIcon,
  ChevronRightSIcon,
  CloseIcon,
  DownloadIcon,
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

import {
  fetchCmsDetail,
  fetchCmsPlay,
  fetchCmsProxy,
  fetchCmsSearch,
  fetchRecBarrage,
  fetchRecMatch,
} from '@/api/film';
import { addHistory, addStar, delStar, findHistory, findStar, putHistory, putStar } from '@/api/moment';
import { fetchAnalyzeActive, fetchParse } from '@/api/parse';
import { setProxy } from '@/api/proxy';
import { cdpSnifferMedia } from '@/api/system';
import LazyBg from '@/components/lazy-bg/index.vue';
import { mediaUtils } from '@/components/multi-player';
import SharePopup from '@/components/share/index.vue';
import TagNav from '@/components/tag-nav/index.vue';
import TitleMenu from '@/components/title-menu/index.vue';
import { emitterChannel } from '@/config/emitterChannel';
import type { IStorePlayer } from '@/config/player';
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

const emits = defineEmits(['update', 'barrage', 'create', 'pause', 'seek']);

// ==================== 推送链接辅助函数 ====================
// 获取 push_agent 站点配置（使用 toRaw 解包 Proxy）
const getPushAgentSite = () => {
  const rawExtra = toRaw(props.store.data.extra);
  return rawExtra?.site?.find((s: any) => s.key === 'push_agent');
};

// 规范化推送链接：移除 push:// 前缀并进行 URL 解码
const normalizePushUrl = (url: string): string => {
  if (url.startsWith('push://')) {
    return decodeURIComponent(url.substring(7));
  }
  return url;
};

// 请求推送详情，获取真实剧集列表
const fetchPushEpisodes = async (pushUrl: string): Promise<{ text: string; link: string }[] | null> => {
  const pushSite = getPushAgentSite();
  if (!pushSite) {
    console.error('[Push] 未找到 push_agent 站点');
    return null;
  }
  const normalizedUrl = normalizePushUrl(pushUrl);
  console.log('[Push] 请求详情，URL:', normalizedUrl);
  const resp = await fetchCmsDetail({ uuid: pushSite.id, ids: normalizedUrl });
  if (resp?.list?.length && resp.list[0]?.vod_episode) {
    const playList = resp.list[0].vod_episode;
    const firstFlag = Object.keys(playList)[0];
    const episodes = playList[firstFlag];
    if (episodes && episodes.length) {
      // 为每个剧集添加标识，方便用户识别来源
      return episodes.map((ep: any) => ({
        ...ep,
        text: `${ep.text}`,
      }));
    }
  }
  return null;
};
// ==================== 辅助函数结束 ====================

const infoConf = ref(props.store.data.info as ICmsInfo);
const extraConf = ref(props.store.data.extra);
const playerConf = ref(props.store.setting);
const processConf = ref(props.process);

const lineList = ref<{ type_id: string; type_name: string }[]>([]);
const analyzeConfig = ref({
  default: {} as IModels['analyze'],
  list: [] as IModels['analyze'][],
});

const starData = ref({} as IModels['star']);
const historyData = ref({} as IModels['history']);
const videoData = ref<IVideoOptions>({
  url: '',
  playEnd: false,
  watchTime: 0,
  duration: 0,
  skipTimeInStart: 30,
  skipTimeInEnd: 30,
});
const recommendList = ref<IRecMatch[]>([]);

const downloadFormData = ref({
  episode: {} as ICmsInfo['vod_episode'],
  current: '',
});
const shareFormData = ref({
  name: '',
  url: '',
  enablePrefix: true,
});
const settingFormData = ref({
  skipHeadAndEnd: false,
  skipTimeInStart: 30,
  skipTimeInEnd: 30,
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
});

const preload = ref({
  status: 'idle' as 'idle' | 'ready' | 'loading' | 'error',
  id: '',
  url: '',
  quality: [] as Array<string | number>,
  headers: {} as Record<string, any>,
  mediaType: '',
});

const navOptions = computed(() => [
  { value: 'episode', label: t('pages.player.film.anthology') },
  ...(recommendList.value.length ? [{ value: 'recommend', label: t('pages.player.film.recommend') }] : []),
]);

const activeAnalyzeList = computed(() => {
  const flag = active.value.filmSource;
  const resp = analyzeConfig.value.list.filter((item: IModels['analyze']) => (item.flag || []).includes(flag));
  return resp;
});

const activeSessionList = computed(() => {
  const flag = active.value.filmSource;
  const resp = infoConf.value.vod_episode?.[flag] || [];
  return resp;
});

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
  (val) => (processConf.value = val),
);
watch(
  () => processConf.value,
  (val) => active.value.watch && timerUpdatePlayProcess(val.currentTime, val.duration),
);

const renderDefaultLazy = () => <LazyBg class="render-icon" />;

onMounted(() => setup());

const createStarDoc = (item: ICmsInfo, siteKey: string) => ({
  type: 1,
  relateId: siteKey,
  videoId: item.vod_id,
  videoImage: item.vod_pic,
  videoName: item.vod_name,
  videoType: item.type_name,
  videoRemarks: item.vod_remarks,
});

const getStarData = async () => {
  try {
    const resp = await findStar({ relateId: extraConf.value.active.key, videoId: infoConf.value.vod_id, type: 1 });
    starData.value = isNil(resp?.id) ? {} : resp;
  } catch (error) {
    console.error('Get Star Data Error:', error);
    starData.value = {} as IModels['star'];
  }
};

const saveStarData = async () => {
  const id = starData.value?.id;
  const doc = createStarDoc(infoConf.value as ICmsInfo, extraConf.value.active.key);

  try {
    const resp = isNil(id) ? await addStar(doc) : await putStar({ id: [id], doc });
    if (isArray(resp) && !isArrayEmpty(resp) && !isNil(resp[0]?.id)) {
      starData.value = resp[0];
    } else {
      starData.value = {} as IModels['star'];
    }
  } catch (error) {
    console.error('Save Star Data Error:', error);
    starData.value = {} as IModels['star'];
  }
};

const delStarDate = async () => {
  const id = starData.value?.id;

  try {
    await delStar({ id: [id] });
  } catch (error) {
    console.error('Delete Star Data Error:', error);
  } finally {
    starData.value = {} as IModels['star'];
  }
};

const handleSwitchStar = async () => {
  const id = starData.value?.id;

  isNil(id) ? await saveStarData() : await delStarDate();
};

const createHistoryDoc = (item: ICmsInfo, siteKey: string, videoData: IVideoOptions) => ({
  type: 1,
  relateId: siteKey,
  siteSource: active.value.filmSource,
  playEnd: false,
  videoId: item.vod_id,
  videoImage: item.vod_pic,
  videoName: item.vod_name,
  videoIndex: active.value.filmIndex,
  watchTime: isPositiveFiniteNumber(videoData.watchTime) ? videoData.watchTime : 0,
  duration: isPositiveFiniteNumber(videoData.duration) ? videoData.duration : 0,
  skipTimeInStart: videoData.skipTimeInStart,
  skipTimeInEnd: videoData.skipTimeInEnd,
});

const getHistoryData = async () => {
  try {
    const resp = await findHistory({ relateId: extraConf.value.active.key, videoId: infoConf.value.vod_id, type: 1 });

    const history = isNil(resp?.id) ? {} : resp;
    historyData.value = history;

    const { skipHeadAndEnd } = playerConf.value;

    const skipTimeInStart = history.skipTimeInStart ?? 30;
    const skipTimeInEnd = history.skipTimeInEnd ?? 30;
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
  } catch (error) {
    console.error('[player][getHistoryData]', error);
    historyData.value = {} as IModels['history'];
  }
};

const saveHistoryData = async () => {
  const id = historyData.value?.id;
  const doc = createHistoryDoc(infoConf.value as ICmsInfo, extraConf.value.active.key, videoData.value);

  try {
    const resp = isNil(id) ? await addHistory(doc) : await putHistory({ id: [id], doc });
    if (isArray(resp) && !isArrayEmpty(resp) && !isNil(resp[0]?.id)) {
      historyData.value = resp[0];
    } else {
      historyData.value = {} as IModels['history'];
    }
  } catch (error) {
    console.error('Save History Data Error:', error);
    historyData.value = {} as IModels['history'];
  }
};

const throttleSaveHistory = throttle(saveHistoryData, 3000, { edges: ['leading', 'trailing'] });

const defaultPreloadConfig = () => {
  preload.value = {
    status: 'idle',
    id: '',
    url: '',
    quality: [],
    headers: {},
    mediaType: '',
  };
};

const handleSwitchSeriesTab = (val: string) => {
  active.value.nav = val;
};

const handleDownloadDialog = () => {
  downloadFormData.value = {
    episode: infoConf.value.vod_episode,
    current: videoData.value.url,
  };
  active.value.download = true;
};

const handleSharePopup = () => {
  let name = infoConf.value.vod_name!;
  const index = active.value.filmIndex.includes('$') ? active.value.filmIndex.split('$')[0] : '';
  if (index) name = `${name}-${index}`;

  shareFormData.value = { ...shareFormData.value, name, url: videoData.value.url };
  active.value.share = true;
};

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

const onSettingChange = (item) => {
  const { skipTimeInStart = 30, skipTimeInEnd = 30, skipHeadAndEnd, playNextPreload, playNextEnabled, skipAd } = item;

  /** sync skip time */
  videoData.value.skipTimeInStart = skipTimeInStart;
  videoData.value.skipTimeInEnd = skipTimeInEnd;

  /** sync player config */
  playerConf.value.skipHeadAndEnd = skipHeadAndEnd;
  playerConf.value.playNextPreload = playNextPreload;
  playerConf.value.playNextEnabled = playNextEnabled;
  playerConf.value.skipAd = skipAd;

  emits('update', { setting: playerConf.value });
};

const reverseOrderIndex = (current: number) => {
  const total = activeSessionList.value.length;
  const type = active.value.reverseOrder;

  if (!isPositiveFiniteNumber(current) || !isPositiveFiniteNumber(total)) return 1;
  if (current >= total) return 1;

  return type ? current + 1 : total - current;
};

const handleSwitchLine = (id: string) => {
  active.value.filmSource = id;
};

const handleSwitchSeason = async (item: ICmsInfoEpisode, index: number) => {
  const currentLine = active.value.filmSource;
  
  // ========== 处理 push:// 网盘推送链接 ==========
  if (item.link && item.link.startsWith('push://')) {
    const loadingMsg = MessagePlugin.loading('正在解析网盘内容，请稍候...', 0);
    
    try {
      const realEpisodes = await fetchPushEpisodes(item.link);
      if (!realEpisodes || realEpisodes.length === 0) {
        throw new Error('解析失败，未获取到剧集列表');
      }
      
      // 获取当前线路的原始剧集数组
      const currentEpisodes = infoConf.value.vod_episode?.[currentLine];
      if (!currentEpisodes || !Array.isArray(currentEpisodes)) {
        throw new Error('当前线路数据异常');
      }
      
      // 关键：替换当前项为真实剧集列表（展开）
      const newEpisodes = [
        ...currentEpisodes.slice(0, index),
        ...realEpisodes,
        ...currentEpisodes.slice(index + 1)
      ];
      
      // 强制触发响应式更新：替换整个 vod_episode 对象
      infoConf.value.vod_episode = {
        ...infoConf.value.vod_episode,
        [currentLine]: newEpisodes
      };
      
      // 同步更新父组件 store
      emits('update', { data: { info: infoConf.value, extra: extraConf.value } });
      
      // 手动更新 lineList（确保 UI 中的线路 Tab 刷新，虽然通常不会变，但保险）
      lineList.value = Object.keys(infoConf.value.vod_episode).map((key) => ({
        type_id: key,
        type_name: key
      }));
      
      // 自动播放替换后的第一集（真实剧集列表的第一个）
      const firstReal = realEpisodes[0];
      active.value.filmIndex = `${firstReal.text}$${firstReal.link}`;
      active.value.watch = false;
      
      // 重置播放进度相关状态（因为是新内容）
      videoData.value = {
        ...videoData.value,
        duration: 0,
        watchTime: playerConf.value.skipHeadAndEnd ? videoData.value.skipTimeInStart : 0,
        playEnd: false,
      };
      
      // 关闭 loading 并播放
      MessagePlugin.close(loadingMsg);
      await callPlay(firstReal);
      return;
      
    } catch (error) {
      MessagePlugin.close(loadingMsg);
      console.error('[Push] 解析失败:', error);
      MessagePlugin.error('解析网盘内容失败，请稍后重试');
      return;
    }
  }
  
  // ========== 原有正常播放逻辑（非 push:// 链接）==========
  active.value.filmIndex = `${item.text}$${item.link}`;
  active.value.watch = false;

  if (active.value.filmIndex !== historyData.value.videoIndex) {
    videoData.value = {
      ...videoData.value,
      duration: 0,
      watchTime: playerConf.value.skipHeadAndEnd ? videoData.value.skipTimeInStart : 0,
      playEnd: false,
    };
  }

  await callPlay(item);
};

const handleSwitchParse = async (id: string) => {
  active.value.analyzeId = id;

  if (active.value.filmIndex) {
    const [text, link] = active.value.filmIndex.split('$');
    await callPlay({ text, link });
  }
};

const reverseOrderEvent = () => {
  infoConf.value.vod_episode = Object.fromEntries(
    Object.entries(infoConf.value.vod_episode!).map(([key, arr]) => [key, arr.toReversed()]),
  );

  active.value.reverseOrder = !active.value.reverseOrder;
};

const getAnalyzeConfig = async () => {
  try {
    const resp = await fetchAnalyzeActive();
    if (resp?.default?.id) {
      analyzeConfig.value.default = resp.default;
      active.value.analyzeId = resp.default.id;
    }
    if (resp?.list) {
      analyzeConfig.value.list = resp.list;
    }
  } catch (error) {
    console.error(`Failed to get analyze config:`, error);
  }
};

const fetchRecommend = async () => {
  try {
    let { vod_name: name, vod_year: year } = infoConf.value;
    if (!year) year = new Date().getFullYear();

    const res = await fetchRecMatch({ name, year });

    recommendList.value = res || [];
  } catch {
    recommendList.value = [];
  }
};

const handleSwitchRecommendItem = async (item: IRecMatch) => {
  const site = extraConf.value.active;

  const searchResp = await fetchCmsSearch({ uuid: site.id, wd: item.vod_name, page: 1 });
  if (!isArray(searchResp.list) || isArrayEmpty(searchResp.list) || isNil(searchResp.list[0]?.vod_id)) {
    MessagePlugin.warning(t('pages.player.message.noRecMatch'));
    return;
  }

  const detailResp = await fetchCmsDetail({ uuid: site.id, ids: searchResp.list[0].vod_id });
  if (
    !isArray(detailResp.list) ||
    isArrayEmpty(detailResp.list) ||
    !isObject(detailResp.list[0]?.vod_episode) ||
    isObjectEmpty(detailResp.list[0]?.vod_episode)
  ) {
    MessagePlugin.warning(t('pages.film.message.noDetailInfo'));
    return;
  }

  const info = {
    ...detailResp.list[0],
    ...(detailResp.list[0]?.vod_id ? {} : { vod_id: searchResp.list[0]?.vod_id }),
    ...(detailResp.list[0]?.vod_name ? {} : { vod_name: searchResp.list[0]?.vod_name }),
    ...(detailResp.list[0]?.vod_pic ? {} : { vod_pic: searchResp.list[0]?.vod_pic }),
  };

  recommendList.value = [];
  historyData.value = {} as IModels['history'];
  starData.value = {} as IModels['star'];
  active.value.reverseOrder = true;
  active.value.nav = 'episode';

  videoData.value = { url: '', playEnd: false, watchTime: 0, duration: 0, skipTimeInStart: 30, skipTimeInEnd: 30 };

  await emits('update', {
    data: toRaw({ info, extra: extraConf.value }),
  });
  setup();
};

const setup = async () => {
  const episode = infoConf.value.vod_episode;
  if (!isObject(episode) || isObjectEmpty(episode)) return;

  const episodeKeys = Object.keys(episode);
  let filmSource = episodeKeys[0];
  let flimEpisode = episode[filmSource]?.[0];

  if (!isObject(flimEpisode) || isObjectEmpty(flimEpisode)) return;
  let filmIndex = `${flimEpisode.text}$${flimEpisode.link}`;

  lineList.value = episodeKeys.map((key) => ({ type_id: key, type_name: key }));

  await getHistoryData();
  if (historyData.value.siteSource && episode[historyData.value.siteSource]) {
    filmSource = historyData.value.siteSource;
  }
  if (historyData.value.videoIndex) filmIndex = historyData.value.videoIndex;
  active.value.filmSource = filmSource;
  active.value.filmIndex = filmIndex;

  flimEpisode = { text: filmIndex.split('$')[0], link: filmIndex.split('$')[1] };

  await getAnalyzeConfig();
  
  // 关键：检查当前要播放的剧集是否为 push:// 链接
  const currentFirstLink = infoConf.value.vod_episode[filmSource]?.[0]?.link;
  if (currentFirstLink && currentFirstLink.startsWith('push://')) {
    // 不自动播放，等待用户点击剧集
    // 但是仍然需要获取收藏和推荐数据
    getStarData();
    fetchRecommend();
    // 可选：显示一个提示
    MessagePlugin.info('检测到网盘推送链接，请点击播放', 3000);
  } else {
    // 正常播放第一集（非推送链接）
    await callPlay(flimEpisode);
    getStarData();
    fetchRecommend();
  }
};

// 辅助函数：处理 push:// 链接，移除前缀并解码
const normalizePushUrl = (url: string): string => {
  if (url.startsWith('push://')) {
    // 移除 push:// 前缀
    const rawUrl = url.substring(7);
    // URL 解码
    return decodeURIComponent(rawUrl);
  }
  return url;
};

const getDirectPlayUrl = async (
  item: ICmsInfoEpisode,
): Promise<{
  url: string;
  headers: Record<string, any>;
  quality: Array<string | number>;
  mediaType: string;
}> => {
  // 辅助递归函数
  const resolvePlayUrl = async (
    playItem: ICmsInfoEpisode,
    overrideSiteId?: string,
    overrideFlag?: string,
  ): Promise<{
    url: string;
    headers: Record<string, any>;
    quality: Array<string | number>;
    mediaType: string;
  }> => {
    // ========== 处理 push:// 网盘推送链接 ==========
    if (playItem.link && playItem.link.startsWith('push://')) {
      // 规范化链接：移除 push:// 前缀并解码
      const normalizedLink = normalizePushUrl(playItem.link);
      console.log('[Push] 规范化后的链接:', normalizedLink);
      
      const rawExtra = toRaw(props.store.data.extra);
      const pushSite = rawExtra?.site?.find((s: any) => s.key === 'push_agent');
      if (!pushSite) {
        throw new Error('未配置网盘推送源(push_agent)');
      }
      
      // 使用规范化后的链接请求详情接口
      const resp = await fetchCmsDetail({ uuid: pushSite.id, ids: normalizedLink });
      console.log('[Push] 详情接口响应:', resp);
      
      if (resp?.list?.length && resp.list[0]?.vod_episode) {
        const playList = resp.list[0].vod_episode;
        // 获取第一个线路（网盘名称）
        const firstFlag = Object.keys(playList)[0];
        const firstEpisode = playList[firstFlag]?.[0];
        
        if (firstEpisode?.link) {
          // 注意：firstEpisode.link 可能是相对路径或加密字符串，不需要再处理 push://
          console.log('[Push] 获取到第一个剧集，link:', firstEpisode.link);
          
          // 重要：更新当前激活的线路和剧集索引，让 UI 刷新
          active.value.filmSource = firstFlag;
          active.value.filmIndex = `${firstEpisode.text}$${firstEpisode.link}`;
          
          // 递归调用，传入 push_agent 的站点 ID 和线路标识
          return resolvePlayUrl(firstEpisode, pushSite.id, firstFlag);
        }
      }
      throw new Error('获取网盘播放列表失败');
    }

    // 正常播放流程
    const siteId = overrideSiteId || extraConf.value.active.id;
    const flag = overrideFlag || active.value.filmSource;

    // 规范化链接（如果是 push:// 但之前没处理，这里再处理一次）
    const playLink = playItem.link.startsWith('push://') ? normalizePushUrl(playItem.link) : playItem.link;
    console.log('[Push] 调用播放接口，siteId:', siteId, 'flag:', flag, 'link:', playLink);

    const playRes = await fetchCmsPlay({
      uuid: siteId,
      play: playLink,
      flag: flag,
    });

    if (!playRes.url) throw new Error('No Play URL');

    const checkPlayable = async (
      url: string,
      headers: Record<string, any> = {},
    ): Promise<{
      url: string;
      headers: Record<string, any>;
      quality: Array<string | number>;
      mediaType: string;
    } | null> => {
      const mediaType = await mediaUtils.checkMediaType(url, headers);
      if (mediaType === 'unknown') return null;
      return { url, headers, mediaType, quality: [] };
    };

    // Direct play
    if (playRes.parse === 0 && playRes.jx !== 1) {
      if (playRes.url.startsWith(PROXY_API)) {
        const { searchParams } = new URL(playRes.url);
        const proxyParams = Object.fromEntries(searchParams.entries());
        const proxyData = await fetchCmsProxy({ uuid: extraConf.value.active.id, ...proxyParams });
        await setProxy({ url: proxyParams.url, text: proxyData });
      }
      const directed = await checkPlayable(playRes.url, playRes.headers);
      if (!isNil(directed)) return { ...directed, quality: playRes.quality };
    }

    // Parse play
    if (playRes.parse === 1 && playRes.jx !== 1) {
      const parsed = await checkPlayable(playRes.url, playRes.headers);
      if (!isNil(parsed)) return { ...parsed, quality: playRes.quality };
    }

    // Jx play
    if (playRes.jx === 1 || !isArrayEmpty(activeAnalyzeList.value)) {
      const parse = activeAnalyzeList.value.find((p: IModels['analyze']) => p.id === active.value.analyzeId);
      if (isNil(parse)) throw new Error('No Active Analyze');
      const jxResp = await fetchParse({ id: parse.id, url: playRes.url });
      const jxed = await checkPlayable(jxResp.url, jxResp.headers);
      if (jxed) return jxed;
    }

    // Sniffer play
    if (isHttp(playRes.url)) {
      const sniffResp = await cdpSnifferMedia({
        url: playRes.url,
        options: {
          runScript: playRes.script.runScript,
          initScript: playRes.script.initScript,
          customRegex: playRes.script.customRegex,
          snifferExclude: playRes.script.snifferExclude,
          headers: playRes.headers,
        },
      });
      const sniffed = await checkPlayable(sniffResp.url, playRes.headers);
      if (!isNil(sniffed)) return sniffed;
    }

    throw new Error('No Play URL');
  };

  return resolvePlayUrl(item);
};

const callBarrage = async (item: ICmsInfoEpisode) => {
  try {
    const res = await fetchRecBarrage({ id: item.link });
    if (!isArray(res.list) || isArrayEmpty(res.list)) return;
    emits('barrage', { list: res.list, id: res.id });
  } catch {}
};


const callPlay = async (item: ICmsInfoEpisode) => {
  try {
    const isPreload = preload.value.status === 'ready' && preload.value.id === item.link;

    const res = isPreload
      ? {
          url: preload.value.url,
          quality: preload.value.quality,
          headers: preload.value.headers,
          mediaType: preload.value.mediaType,
        }
      : await getDirectPlayUrl(item);

    videoData.value.url = res.url;
    if (isPreload) videoData.value.watchTime = playerConf.value.skipHeadAndEnd ? videoData.value.skipTimeInStart : 0;

    await emits('create', {
      url: res.url,
      quality: res.quality,
      headers: res.headers,
      startTime: videoData.value.watchTime,
      skipAd: playerConf.value.skipAd,
      next: !getEpisodePlayState()?.isLast,
    });
    callBarrage(item);

    active.value.watch = true;
  } catch (error) {
    console.error(`[player][callPlay][error]`, error);

    const msg = (error as Error).message;
    if (msg === 'No Play URL') {
      MessagePlugin.warning(t('pages.player.message.noPlayUrl'));
    } else if (msg === 'No Active Analyze') {
      MessagePlugin.warning(t('pages.player.message.noActiveAnalyze'));
    } else {
      MessagePlugin.error(`${t('common.error')}: ${msg}`);
    }
  } finally {
    defaultPreloadConfig();
  }
};

const timerUpdatePlayProcess = async (currentTime: number, duration: number) => {
  if (!isPositiveFiniteNumber(currentTime) || !isPositiveFiniteNumber(duration)) return;
  if (currentTime <= 0 || duration <= 0) return;

  const state = getEpisodePlayState();
  if (!state) return;

  const { skipTimeInStart, skipTimeInEnd } = videoData.value;
  const { skipHeadAndEnd, playNextEnabled, playNextPreload } = playerConf.value;

  /** skip head */
  if (skipHeadAndEnd && skipTimeInStart < duration && currentTime < skipTimeInStart) {
    emits('seek', skipTimeInStart);
    return;
  }

  /** unified watch time */
  const watchTime = skipHeadAndEnd ? currentTime + skipTimeInEnd : currentTime;
  const isPlayEnd = watchTime >= duration;

  // console.debug(`timeUpdate`, {
  //   currentTime,
  //   watchTime,
  //   duration,
  //   percentage: Math.trunc((currentTime / duration) * 100),
  // });

  /** update history */
  videoData.value = { ...videoData.value, watchTime, duration, playEnd: isPlayEnd };
  throttleSaveHistory();

  /** play next episode */
  if (isPlayEnd && playNextEnabled && !state.isLast) {
    emits('pause');
    await handleSwitchSeason(state.nextInfo);
    return;
  }

  /** preload next episode */
  const PRELOAD_TIME = 30;
  const preloadWatchTime = skipHeadAndEnd ? currentTime + PRELOAD_TIME + skipTimeInEnd : currentTime + PRELOAD_TIME;
  if (playNextPreload && !state.isLast && preload.value.status === 'idle' && preloadWatchTime >= duration) {
    preload.value.status = 'loading';

    try {
      const nextd = await getDirectPlayUrl(state.nextInfo);
      if (isNil(nextd)) return;

      preload.value = {
        ...preload.value,
        ...nextd,
        id: state.nextInfo.link,
        status: 'ready',
      };
    } catch (error) {
      preload.value.status = 'error';
      console.error(`[player][timeUpdate][preloadNext][error]`, error);
    }
  }
};

const getEpisodePlayState = (): {
  currIndex: number;
  nextIndex: number;
  isLast: boolean;
  reverseOrder: boolean;
  currentInfo: ICmsInfoEpisode;
  nextInfo: ICmsInfoEpisode;
} | null => {
  const { filmSource, filmIndex, reverseOrder } = active.value;
  const episode = infoConf.value.vod_episode;

  if (
    isNil(filmSource) ||
    isNil(filmIndex) ||
    !isBoolean(reverseOrder) ||
    !isObject(episode) ||
    isObjectEmpty(episode)
  ) {
    return null;
  }

  const currentEpisode = episode?.[filmSource];
  if (!isArray(currentEpisode) || isArrayEmpty(currentEpisode)) {
    return null;
  }

  const index = currentEpisode.findIndex((item) => `${item.text}$${item.link}` === filmIndex);
  if (index === -1) return null;

  const currentInfo = currentEpisode[index];
  const isLast = reverseOrder ? index === currentEpisode.length - 1 : index === 0;
  const nextIndex = isLast ? -1 : reverseOrder ? index + 1 : index - 1;
  const nextInfo = isLast ? ({} as ICmsInfoEpisode) : currentEpisode[nextIndex];

  return { currIndex: index, nextIndex, isLast, reverseOrder, currentInfo, nextInfo };
};

emitter.on(emitterChannel.COMP_MULTI_PLAYER_PLAYNEXT, async () => {
  const state = getEpisodePlayState();
  if (!isBoolean(state?.isLast) || isNil(state?.nextIndex) || state?.nextIndex === -1) return;

  const nextInfo = state.nextInfo;
  await handleSwitchSeason(nextInfo);
});
</script>
<style lang="less" scoped></style>
