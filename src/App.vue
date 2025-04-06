<script setup lang="ts">
import { ref } from 'vue';
import { useUserStore } from '@/stores/user';
import { useServerStore } from '@/stores/server';
import DXRating from '@/components/DXRating.vue';
import QRCode from '@/components/QRCode.vue';
import CharaInfo from '@/components/CharaInfo.vue';
import PlayerInfo from '@/components/PlayerInfo.vue';
import { useRouter } from 'vue-router';
import CardBack from '@/components/CardBack.vue';

function extractParams(url:string | URL) {
  // 创建一个 URL 对象
  const urlObj = new URL(url);

  // 获取路径中的 MAID 值
  const pathParts = urlObj.pathname.split('/');
  const maid = pathParts[pathParts.length - 1].split(".")[0]; // 获取路径的最后一部分

  // 获取查询参数中的 l 值
  const lParam = urlObj.searchParams.get('l');

  return {
    maimaiCode: maid,
    timeLimit: lParam
  };
}
const userStore = useUserStore();
const serverStore = useServerStore();
const router = useRouter();
console.log(JSON.parse(JSON.stringify(userStore.userProfile)));
const userProfile = ref<UserProfile>({ preferences: {} } as UserProfile);  // Deep copyJSON.parse(JSON.stringify(userStore.userProfile))
import Yamada from "@/assets/ryo.png";
const r = (image: ImagePublic) => import.meta.env.VITE_URL + "/images/" + image!.id;

const prepareDefaultPreferences = () => {
  var code = extractParams(window.location.href);
  console.log(code);
  userProfile.value!.preferences.character_name ="リョウ"//||= userProfile.value!.preferences.character.name;
  userProfile.value!.preferences.display_name ="pingfanh";
  userProfile.value!.preferences.dx_rating =17000 //||= userProfile.value!.player_rating;
  userProfile.value!.preferences.friend_code ||= "889677537036759"; // this is my friend code
  userProfile.value!.preferences.simplified_code="pingfanh";// ||= userStore.simplifiedCode;
  userProfile.value!.preferences.maimai_version ="Ver.1145" //||= serverStore.serverMessage!.maimai_version;
  userProfile.value!.preferences.maimaiCode = code.maimaiCode;//`http://wq.sys-allnet.cn/qrcode/img/${code.maimaiCode}.png?v`;
  
  userProfile.value!.preferences.timeLimit = secondsToHMS(code.timeLimit);

}

prepareDefaultPreferences();

function secondsToHMS(totalSeconds:any) {

// 计算 UTC 时间的小时、分钟和秒
  const utcHours = Math.floor((totalSeconds % 86400) / 3600); // UTC 小时
  const utcMinutes = Math.floor((totalSeconds % 3600) / 60); // UTC 分钟
  const utcSeconds = totalSeconds % 60; // UTC 秒

// 转换为中国时间（UTC+8）
  let chinaHours = utcHours + 8;
  if (chinaHours >= 24) {
    chinaHours -= 24; // 如果超过 24 小时，减去 24
  }

// 格式化为 HH:MM:SS
  const formattedTime = [
    String(chinaHours).padStart(2, '0'),
    String(utcMinutes).padStart(2, '0'),
    String(utcSeconds).padStart(2, '0')
  ].join(':');
  return formattedTime;
}
</script>
<template>
  <div class="flex items-center justify-center h-full w-full">
    <div class="flex relative flex-col items-center justify-center h-full">
      <CardBack :preferences="userProfile.preferences" />
      <div class="flex flex-col absolute top-0 w-full h-full">
        <div class="header-widget flex relative w-full justify-between">
          <!-- <img class="object-cover w-1/2" :src="r(userProfile?.preferences.passname)"> -->
          <DXRating class="w-1/2" :rating="userProfile.preferences.dx_rating || 0" />
        </div>
        <div class="header-widget flex relative w-full flex-row-reverse">
          <PlayerInfo class="w-1/2" :username="userProfile.preferences.display_name!"
                      :friend-code="userProfile.preferences.friend_code!" />
        </div>
      </div>
      <div class="absolute flex flex-col left-0" style="bottom: 8%;">
        <CharaInfo :chara="userProfile.preferences.character_name!" :time="userProfile.preferences.timeLimit || '12:00:00'" />
      </div>
      <div class="qr-widget absolute" v-if="userProfile.preferences">
<!--        <img :src="userProfile.preferences.maimaiCode" style="width: 40dvw">-->
        <QRCode :content="userProfile.preferences.maimaiCode" :size="userProfile.preferences.qr_size || 23" />
      </div>
      <div class="flex absolute bottom-0 items-center justify-center w-full h-8">
        <div class="footer-widget flex justify-between py-1 rounded-2xl bg-gray-800 text-white opacity-85">
          <p class="footer-text font-sega">{{ userProfile.preferences.simplified_code }}</p>
          <p class="footer-text font-sega">{{ userProfile.preferences.maimai_version }}</p>
        </div>
        <div class="p-1 rounded-full bg-white" @click="">
          <img src="@/assets/misc/settings.svg" style="width: 2vh;"></img>
        </div>
      </div>
    </div>
  </div>
</template>
<style scoped>
.qr-widget {
  bottom: 5%;
  right: 0;
}

.header-widget {
  padding-left: 2%;
  padding-right: 2%;
  padding-top: 0.4%;
}

.footer-widget {
  width: 85%;
  padding-left: 3%;
  padding-right: 3%;
}

.footer-text {
  font-size: 1.2vh;
  line-height: 120%;
}
</style>
