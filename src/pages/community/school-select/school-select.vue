<template>
  <view class="content2">
    <view
      class="d-flex border a-center mx-2 j-sb px-3"
      style="
        border-radius: 70rpx;
        height: 60rpx;
        border-color: #938b8e;
        border-width: 2rpx;
        margin: 10rpx 50rpx 0 50rpx;
      "
    >
      <input
        class="search-text"
        maxlength="10"
        placeholder="输入学校名称或拼音查询"
        type="text"
        value=""
      />
      <image style="width: 40rpx" mode="widthFix" :src="Icons.Search" />
    </view>
    <view class="search-bar">
      <view class="small"> 当前选择</view>
    </view>
    <view class="school-select-box">
      <view class="current_school">
        <view class="current_school_text">{{ currentSchool }}</view>
      </view>
      <view v-if="sel">
        <view class="select" @click="change">
          <view class="text"
            >{{ currentCampus }}
            <image :src="Icons.DownBlack" />
          </view>
        </view>
      </view>
      <view v-else>
        <view class="select2" @click="change">
          <view class="text"
            >{{ currentCampus }}
            <image :src="Icons.UpBlack" />
          </view>
        </view>
        <view class="option" @click="change">
          <view
            v-for="(item, index) in campuses.data"
            :key="index"
            class="text"
            @click="changeCampus(item.name, index)"
          >
            {{ item.name }}
          </view>
        </view>
      </view>
    </view>
    <view class="search-bar">
      <view class="small"> 定位/历史记录</view>
    </view>
    <view
      v-for="(item, index) in historyJSON.histories"
      :key="index"
      class="big"
    >
      <view
        class="bubble"
        @click="
          changeCampusByHistory(
            item.campusName,
            item.schoolName,
            item.communityId,
            item.schoolId
          )
        "
      >
        {{ item.schoolName }} ({{ item.campusName }})</view
      >
    </view>
    <view class="search-bar">
      <view class="small"> 热门学校</view>
    </view>
    <view class="big">
      <view style="margin-right: 50rpx">
        <view
          v-for="(item, index) in schools.data"
          :key="index"
          class="bubble"
          @click="changeSchool(item.name, index)"
        >
          {{ item.name }}
        </view>
      </view>
    </view>
  </view>

  <view class="blank" />

  <view class="content2">
    <view class="small"> 所有学校</view>
    <view
      v-for="(item1, index1) in schools.data"
      :key="index1"
      class="school"
      @click="changeSchool(item1.name, index1)"
    >
      {{ item1.name }}
    </view>
  </view>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import { Community } from "@/apis/schemas";
import { listCommunity } from "@/apis/community/community";
import { onLoad } from "@dcloudio/uni-app";
import { StorageKeys } from "@/utils/const";
import { Icons } from "@/utils/url";

const currentSchool = ref("");
const currentCampus = ref("");
const communityId = ref("");
const parentId = ref("");
let history = reactive({
  campusName: "",
  schoolName: "",
  communityId: "",
  schoolId: ""
});

let historyJSON = reactive({
  histories: reactive<Array<any>>([])
});

function init() {
  communityId.value = uni.getStorageSync(StorageKeys.CommunityId);
  if (!uni.getStorageSync(StorageKeys.CommunityId)) {
    uni.setStorageSync(StorageKeys.CommunityId, "637ce159b15d9764c31f9c84");
  }
  communityId.value = uni.getStorageSync(StorageKeys.CommunityId);
}

function getHistories() {
  if (uni.getStorageSync(StorageKeys.HistoryCampuses)) {
    historyJSON = JSON.parse(
      decodeURIComponent(uni.getStorageSync(StorageKeys.HistoryCampuses))
    );
  }
  if (checkRepeat(history.communityId)) {
    if (historyJSON.histories.length === 3) {
      historyJSON.histories.pop();
      historyJSON.histories.unshift(history);
    } else {
      historyJSON.histories.unshift(history);
    }
  }
  uni.setStorageSync(
    StorageKeys.HistoryCampuses,
    encodeURIComponent(JSON.stringify(historyJSON))
  );
}
function checkRepeat(id: string) {
  let flag = true;
  for (let i = 0; i < historyJSON.histories.length; i++) {
    if (id === historyJSON.histories[i].communityId) {
      flag = false;
    }
  }
  if (id === "") {
    flag = false;
  }
  return flag;
}
onLoad(() => {
  getCampus();
});

const lists = reactive<{
  data: Community[];
}>({
  data: []
});

const campuses = reactive<{
  data: Community[];
}>({
  data: []
});

const schools = reactive<{
  data: Community[];
}>({
  data: []
});

// 所有Community列表
async function schoolList() {
  lists.data = (await listCommunity({})).communities;
}
// 找所有学校
async function getSchools() {
  schoolList().then(async () => {
    let j = 0;
    for (let i = 0; i < lists.data.length; i++) {
      if (!lists.data[i].parentId) {
        schools.data[j] = lists.data[i];
        j++;
      }
    }
  });
}
getSchools();

// 找该学校的所有校区
async function getCampus() {
  if (parentId.value) {
    campuses.data = (
      await listCommunity({
        parentId: parentId.value
      })
    ).communities;
  } else {
    schoolList().then(async () => {
      init();
      for (let i = 0; i < lists.data.length; i++) {
        if (lists.data[i].id === communityId.value) {
          currentCampus.value = lists.data[i].name;
          parentId.value = <string>lists.data[i].parentId;
        }
      }
      for (let j = 0; j < lists.data.length; j++) {
        if (lists.data[j].id === parentId.value) {
          currentSchool.value = lists.data[j].name;
        }
      }
      history.campusName = currentCampus.value;
      history.schoolName = currentSchool.value;
      history.communityId = communityId.value;
      history.schoolId = parentId.value;
      getHistories();
      campuses.data = (
        await listCommunity({
          parentId: parentId.value
        })
      ).communities;
    });
  }
}
getCampus();

const sel = ref(true);

// 更改下拉选框状态
function change() {
  sel.value = !sel.value;
}
// 选择学校
function changeCampus(name: string, index: number) {
  currentCampus.value = name;
  uni.setStorageSync(StorageKeys.CommunityId, campuses.data[index].id);
}
function changeCampusByHistory(
  campus: string,
  school: string,
  id: string,
  parent: string
) {
  sel.value = true;
  currentCampus.value = campus;
  currentSchool.value = school;
  parentId.value = parent;
  uni.setStorageSync("communityId", id);
  getCampus();
}
// 选择校区
function changeSchool(name: string, index: number) {
  currentSchool.value = name;
  parentId.value = schools.data[index].id;
  currentCampus.value = "请选择校区";
  sel.value = false;
  getCampus();
}
</script>

<style lang="scss" scoped>
.top-padding {
  height: calc(58 / 390 * 100vw);
}

.school {
  margin: 30rpx 50rpx 0 50rpx;
  font-size: 25rpx;
  text-align: left;
  display: grid;
  padding-bottom: 15rpx;
  border-bottom: 1px solid #f6f6f6;
}

// 分隔栏
.blank {
  background-color: #f6f6f6;
  margin-top: 40rpx;
  display: flex;
  width: 100%;
  height: 20rpx;
}

.history {
  width: 450rpx;
}

// 气泡
.bubble {
  background-color: #f6f6f6;
  margin-left: 50rpx;
  margin-top: 20rpx;
  font-size: 25rpx;
  text-align: center;
  display: inline-block;
  padding: 20rpx 30rpx 20rpx 30rpx;
  border-radius: 50rpx;
}

// 校区选择
.school-select-box {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
}

.current_school {
  width: 450rpx;
}

.current_school_text {
  margin-left: 50rpx;
  margin-top: 20rpx;
  font-weight: bold;
  border-bottom: 2px solid skyblue;
  display: inline-block;
  font-size: 35rpx;
}

.small {
  height: 40rpx;
  width: 100%;
  display: flex;
  position: relative;
  margin-top: 40rpx;
  margin-left: 50rpx;
  font-size: 25rpx;
  color: #afafaf;
}

.big {
  display: flex;
  width: 100%;
  height: auto;
}

.content1 {
  position: fixed;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.content2 {
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.cancel {
  width: 40rpx;
  height: 40rpx;
  margin: 8rpx 0 30rpx 30rpx;
}

.school-bar {
  display: grid;
  width: 100%;
  margin-top: 2%;
}

// 下拉菜单
.select {
  padding: 10rpx 10rpx 10rpx 20rpx;
  border: black 1px solid;
  border-radius: 15px;
  position: relative;
  margin-top: 15rpx;
  margin-right: 50rpx;
  display: flex;
  align-items: center;
  image {
    width: 26rpx;
    height: 15rpx;
    padding-right: 10rpx;
    margin-top: 8rpx;
    float: right;
  }
  .text {
    font-size: 25rpx;
    font-weight: bold;
    width: 225rpx;
  }
}

.select2 {
  padding: 10rpx 10rpx 10rpx 20rpx;
  border: black 1px solid;
  border-bottom: none;
  border-top-left-radius: 15px;
  border-top-right-radius: 15px;
  position: relative;
  margin-top: 15rpx;
  margin-right: 50rpx;
  display: flex;
  align-items: center;
  image {
    width: 26rpx;
    height: 15rpx;
    padding-right: 10rpx;
    margin-top: 8rpx;
    float: right;
  }
  .text {
    font-size: 25rpx;
    font-weight: bold;
    width: 225rpx;
  }
}

.option {
  padding: 2rpx 10rpx 2rpx 20rpx;
  border: black 1px solid;
  border-top: none;
  border-bottom-right-radius: 15px;
  border-bottom-left-radius: 15px;
  position: absolute;
  line-height: 60rpx;
  margin-right: 20rpx;
  z-index: 100;

  .text {
    color: #7f7f81;
    font-size: 25rpx;
    font-weight: bold;
    width: 225rpx;
    position: relative;
    display: flex;
  }

  .text::before {
    content: "";
    position: absolute;
    width: 98%;
    height: 1rpx;
    background-color: #7f7f81;
  }
}
</style>
