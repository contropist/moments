<template>

  <div class="memo flex flex-row gap-2 sm:gap-4 text-sm border-x-0 pt-2 p-2 sm:p-4"
    :class="{ 'bg-slate-100 dark:bg-neutral-800': props.memo.pinned }">
    <img :src="props.memo.user.avatarUrl" class="avatar w-9 h-9 rounded cursor-pointer" @click="toDetail" />
    <div class="flex flex-col gap-.5 flex-1">
      <div class="flex flex-row justify-between items-center">
        <div class="username text-[#576b95] mb-1 dark:text-white cursor-pointer" @click="toDetail">{{
          props.memo.user.nickname }}</div>
        <Pin :size=14 v-if="props.memo.pinned" />
      </div>

      <div class="memo-content text-sm friend-md mome-container text-wrap break-all" ref="el" v-html="memoContent">
      </div>
      <div class="flex gap-1">
        <div @click="navigateTo({ query: { tag: tag.substring(1) } })" v-for="tag in tags"
          class="text-[#3C4F7E] cursor-pointer hover:text-[#3C4F7E]/80">{{ tag }}</div>
      </div>

      <div class="flex flex-row gap-2 my-2 bg-[#f7f7f7] dark:bg-[#212121] items-center p-2 border rounded"
        v-if="props.memo.externalFavicon && props.memo.externalTitle">
        <img class="w-8 h-8" :src="props.memo.externalFavicon" alt="">
        <a :href="props.memo.externalUrl" target="_blank" class="text-[#576b95]">{{ props.memo.externalTitle }}</a>
      </div>

      <!--      <iframe class="rounded" frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86-->
      <!--        :src="props.memo.music163Url" v-if="props.memo.music163Url"></iframe>-->

      <div style="max-width: 100%"
        v-if="props.memo.music163Url && props.memo.music163Url !== '' && musicType && musicId">
        <ClientOnly><meting-js :server="musicPlatform" :type="musicType" :id="musicId" :list-folded="true" />
        </ClientOnly>
      </div>

      <iframe class="w-full h-[250px] my-2" v-if="props.memo.bilibiliUrl" :src="props.memo.bilibiliUrl" scrolling="no"
        border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

      <iframe class="w-full h-[250px] my-2" v-if="memoExt.youtubeUrl" :src="memoExt.youtubeUrl" scrolling="no"
        border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>

      <audio class="w-full my-2" :src="memoExt.audioUrl" controls v-if="memoExt.audioUrl"></audio>

      <video class="w-2/3 my-2 rounded" controls v-if="memoExt.videoUrl" preload="metadata">
        <source :src="`${memoExt.videoUrl}#t=1`" type="video/mp4" />
      </video>
      <video class="w-2/3 my-2 rounded" controls v-if="memoExt.localVideoUrl" preload="metadata">
        <source :src="`${memoExt.localVideoUrl}#t=1`" type="video/mp4" />
      </video>

      <DoubanBook :book="memoExt.doubanBook" v-if="memoExt.doubanBook" />
      <DoubanMovie :movie="memoExt.doubanMovie" v-if="memoExt.doubanMovie" />

      <div v-if="imgs.length">
        <FancyBox class="grid my-1 gap-0.5" :style="gridStyle" :options="{ Carousel: { infinite: false } }">
          <img loading="lazy" :src="getImgUrl(img)" class="cursor-zoom-in rounded"
            :class="imgs.length === 1 ? 'full-cover-image-single' : 'full-cover-image-mult'"
            v-for="(img, index) in imgs" :key="index" />
        </FancyBox>
      </div>

      <div class="text-[#576b95] cursor-pointer" v-if="!isDetailPage && hh > maxHeight && !showAll" @click="showMore">全文
      </div>

      <div class="text-[#576b95] cursor-pointer " v-if="!isDetailPage && showAll" @click="showLess">收起</div>
      <div class="text-[#576b95] font-medium dark:text-white text-xs mt-2 mb-1 select-none" v-if="props.memo.location">
        {{ props.memo.location?.split(/\s+/g).join(' · ') }}</div>
      <div class="toolbar relative flex flex-row justify-between select-none my-1 items-center">
        <div v-if="memo.showType === 0" class="text-xs text-stone-400 mr-2">私密</div>
        <div class="flex-1 text-gray text-xs text-[#9DA4B0] " v-if="publicConfig.dateTimeFormat === 'AGO'">{{
          dayjs(props.memo.createdAt).locale('zh-cn').fromNow().replaceAll(/\s+/g,
            '') }}</div>
        <div class="flex-1 text-gray text-xs text-[#9DA4B0] " v-else>{{
          dayjs(props.memo.createdAt).format('YYYY-MM-DD HH:mm:ss') }}</div>
        <div @click="showToolbar = !showToolbar"
          class="toolbar-icon px-2 py-1 bg-[#f7f7f7] dark:bg-slate-700 hover:bg-[#dedede] cursor-pointer rounded flex items-center justify-center">
          <img src="~/assets/img/dian.svg" class="w-3 h-3" />
        </div>
        <div class="absolute top-[-8px] right-[32px] bg-[#4c4c4c] rounded text-white p-2 px-4" v-if="showToolbar"
          ref="toolbarRef">
          <div class="flex flex-row gap-4">
            <div class="flex flex-row gap-1 cursor-pointer items-center" v-if="token"
              @click="pinned(); showToolbar = false">
              <Pin :size=14 />
              <div class="hidden md:block">{{ (props.memo.pinned ? '取消' : '') + '置顶' }}</div>
            </div>
            <div class="flex flex-row gap-1 cursor-pointer items-center"
              v-if="token && !route.path.startsWith('/detail')" @click="editMemo">
              <FilePenLine :size=14 />
              <div class="hidden md:block">编辑</div>
            </div>
            <AlertDialog>
              <AlertDialogTrigger asChild>
                <div class="flex flex-row gap-1 cursor-pointer items-center" v-if="token">
                  <Trash2 :size=14 />
                  <div class="hidden md:block">删除</div>
                </div>
              </AlertDialogTrigger>
              <AlertDialogContent>
                <AlertDialogHeader>
                  <AlertDialogTitle>确定删除吗?</AlertDialogTitle>
                  <AlertDialogDescription>
                    无法恢复,你确定删除吗?
                  </AlertDialogDescription>
                </AlertDialogHeader>
                <AlertDialogFooter>
                  <AlertDialogCancel>取消</AlertDialogCancel>
                  <AlertDialogAction @click="removeMemo">确定</AlertDialogAction>
                </AlertDialogFooter>
              </AlertDialogContent>
            </AlertDialog>
            <span class="bg-[#6b7280] h-[20px] w-[1px] block md:hidden" v-if="token"></span>
            <div class="flex flex-row gap-1 cursor-pointer items-center" @click="like">
              <Heart :size=14 v-if="likeList.findIndex((id) => id === props.memo.id) < 0" />
              <HeartCrack :size=14 v-else />
              <div>{{ likeList.findIndex((id) => id === props.memo.id) >= 0 ? '取消' : '赞' }}</div>
            </div>
            <span class="bg-[#6b7280] h-[20px] w-[1px]" v-if="!token"></span>
            <div class="flex flex-row gap-1 cursor-pointer items-center" v-if="publicConfig.enableComment"
              @click="momentsShowCommentInput = !momentsShowCommentInput; showUserCommentArray = []; showToolbar = false">
              <MessageSquareMore :size=14 />
              <div>评论</div>
            </div>
          </div>
        </div>
      </div>
      <div class="rounded bottom-shadow bg-[#f7f7f7] dark:bg-[#202020] flex flex-col gap-1  ">
        <div class="flex flex-row py-2 px-4 gap-2 items-center text-sm" v-if="props.memo.favCount > 0">
          <Heart :size=14 color="#C64A4A" />
          <div class="text-[#576b95]"><span class="mx-1">{{ props.memo.favCount }}</span>位访客</div>
        </div>
        <FriendsCommentInput :memoId="props.memo.id" @commentAdded="refreshComment" v-if="momentsShowCommentInput" />
        <template v-if="props.memo.comments.length > 0 && publicConfig.enableShowComment">
          <div class="px-4 py-2 flex flex-col gap-1">
            <div class="relative flex flex-col gap-2 text-sm" v-for="(comment, index) in props.memo.comments"
              :key="index">
              <Comment :comment="comment" @memo-update="refreshComment" :index="index"
                @comment-started="momentsShowCommentInput = false" />
            </div>
            <div v-if="props.memo._count.comments > 5 && props.showMore" class="text-[#576b95] cursor-pointer"
              @click="toDetail">查看更多...</div>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { Memo, MemoExt, PublicConfig } from '@/lib/types';
import { useElementSize, onClickOutside, watchOnce, useStorage } from '@vueuse/core';
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import 'dayjs/locale/zh-cn';
import { Heart, HeartCrack, MessageSquareMore, Trash2, FilePenLine, Pin } from 'lucide-vue-next'
import { memoUpdateEvent } from '@/lib/event'
import { toast } from 'vue-sonner';
import { getImgUrl } from '~/lib/utils';

const token = useCookie('token')
dayjs.extend(relativeTime)
const props = withDefaults(
  defineProps<{
    memo: Memo,
    showMore: boolean,
    index?: number

  }>(), {}
)
const publicConfig = useState<PublicConfig>('publicConfig')
const route = useRoute()
const isDetailPage = route.fullPath.startsWith('/detail')
const emit = defineEmits(['memo-update'])
const maxLine = publicConfig.value.memoMaxLine
const maxHeight = ref(24 * maxLine)


const showAll = ref(false)
const showToolbar = ref(false)
const momentsShowCommentInput = ref(false)
const toolbarRef = ref(null)
const showUserCommentArray = useState<Array<boolean>>('showUserCommentArray_' + props.memo.id, () => [])
const el = ref<any>(null)
let hh = ref(0)

const musicType = ref('')
const musicId = ref('')
const musicPlatform = ref('netease')

if (props.memo.music163Url) {
  if (props.memo.music163Url.includes("music.163.com")) {
    if (props.memo.music163Url.includes("playlist")) {
      musicType.value = 'playlist'
      musicId.value = props.memo.music163Url.split('playlist?id=')[1].split('&')[0]
    } else if (props.memo.music163Url.includes("song")) {
      musicType.value = 'song'
      musicId.value = props.memo.music163Url.split('song?id=')[1].split('&')[0]
    } else if (props.memo.music163Url.includes("album")) {
      musicType.value = 'album'
      musicId.value = props.memo.music163Url.split('album?id=')[1].split('&')[0]
    }
  } else if (props.memo.music163Url.includes("y.qq.com")) {
    musicPlatform.value = 'tencent'
    if (props.memo.music163Url.includes("songDetail")) {
      musicType.value = 'song'
      musicId.value = props.memo.music163Url.split('songDetail/')[1].split('?')[0]
    } else if (props.memo.music163Url.includes("playlist")) {
      musicType.value = 'playlist'
      musicId.value = props.memo.music163Url.split('playlist/')[1].split('?')[0]
    }
  } else {
    props.memo.music163Url = ''
  }
}


const { height } = useElementSize(el)
const likeList = useStorage<Array<number>>('likeList', [])

const linkReg = /\[(.*?)\]\((.*?)\)/g

const memoExt = computed(() => JSON.parse(props.memo.ext || '{}') as MemoExt)
const tags = computed(() => props.memo.content.match(/#(\S+)/g) || [])
const memoContent = computed(() => props.memo.content.replaceAll(/\n/g, '<br/>').replace(/#(\S+)/g, '').replace(linkReg, "<a class='mx-0.5 text-primary/80' href='$2' target='_blank'>$1</a>"))

const imgs = computed(() => props.memo.imgs ? props.memo.imgs.split(',') : []);
const gridStyle = computed(() => {
  let style = 'align-items: start;'; // 确保内容顶部对齐
  switch (imgs.value.length) {
    case 1:
      style += 'grid-template-columns: 1fr;';
      break;
    case 2:
      style += 'grid-template-columns: 1fr 1fr; aspect-ratio: 2 / 1;';
      break;
    case 3:
      style += 'grid-template-columns: 1fr 1fr 1fr; aspect-ratio: 3 / 1;';
      break;
    case 4:
      style += 'grid-template-columns: 1fr 1fr; aspect-ratio: 1;';
      break;
    default:
      style += 'grid-template-columns: 1fr 1fr 1fr; aspect-ratio: 3 / 1;';
  }
  return style;
});

const like = async () => {
  showToolbar.value = false
  const contain = likeList.value.find((id) => id === props.memo.id)
  const res = await $fetch<{ success: boolean }>('/api/memo/like', {
    method: 'POST',
    body: JSON.stringify({
      memoId: props.memo.id,
      like: !contain
    })
  })
  if (res.success) {
    if (contain) {
      likeList.value = likeList.value.filter((id) => id !== props.memo.id)
    } else {
      likeList.value.push(props.memo.id)
    }
    emit('memo-update')
  }
}

const toDetail = () => {
  if (props.showMore) navigateTo(`/detail/${props.memo.id}`);
}

const pinned = async () => {
  showToolbar.value = false
  const res = await $fetch<{ success: boolean }>('/api/memo/pinned', {
    method: 'POST',
    body: JSON.stringify({
      memoId: props.memo.id,
      pinned: !(props.memo.pinned)
    })
  })
  if (res.success) {
    toast.success('操作成功')
    emit('memo-update')
  }
}

const removeMemo = async () => {
  showToolbar.value = false
  const res = await $fetch<{ success: boolean }>('/api/memo/remove', {
    method: 'POST',
    body: JSON.stringify({
      memoId: props.memo.id
    })
  })
  if (res.success) {
    toast.success('删除成功')
    emit('memo-update')
  }
}

const editMemo = async () => {
  showToolbar.value = false
  memoUpdateEvent.emit({ ...props.memo, index: props.index })
}


const refreshComment = async () => {
  emit('memo-update', props.memo)
  showUserCommentArray.value = []
  momentsShowCommentInput.value = false
}



onClickOutside(toolbarRef, () => showToolbar.value = false)


const showMore = () => {
  showAll.value = true
  el.value.classList.remove(`line-clamp-${maxLine}`)
}
const showLess = () => {
  showAll.value = false
  el.value.classList.add(`line-clamp-${maxLine}`)
}



watchOnce(height, () => {
  hh.value = height.value
  if (!isDetailPage && height.value > maxHeight.value) {
    el.value.classList.add(`line-clamp-${maxLine}`)
  }
})

</script>

<style scoped>
.full-cover-image-mult {
  object-fit: cover;
  object-position: center;
  width: 100%;
  aspect-ratio: 1 / 1;
  border: transparent 1px solid;
}

.full-cover-image-single {
  object-fit: cover;
  object-position: center;
  max-height: 300px;
  height: auto;
  width: auto;
  border: transparent 1px solid;
}

.aplayer-body {
  max-width: 100%;
  width: 100%;
}

.aplayer-pic {
  z-index: 1;
}

.aplayer-music {
  overflow: hidden;
  display: inline-block;
  align-items: center;
  width: 100%;
  position: absolute;
  animation: scroll 8s linear infinite;
}

.aplayer-title,
.aplayer-author {
  padding-right: 10px;
}

@keyframes scroll {
  from {
    transform: translateX(100%);
  }

  to {
    transform: translateX(-100%);
  }
}

.aplayer-lrc {
  margin-top: 25px !important;
}
</style>
