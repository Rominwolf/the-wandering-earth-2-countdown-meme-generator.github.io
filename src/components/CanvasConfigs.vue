<template>
  <q-drawer side="right" v-model="drawerRight" bordered
    :width="this.$q.screen.width * 0.2" :breakpoint="500"
            class="font-zihun bg-white q-px-md q-py-md">

    <q-card-actions align="right" class="q-pa-none">
      <q-btn-dropdown color="primary" label="导出选项">
        <q-list>
          <q-item clickable v-close-popup @click="exportImage('jpeg')">
            <q-item-section>
              <q-item-label>下载缩略图 (JPEG)</q-item-label>
            </q-item-section>
          </q-item>
          <q-item clickable v-close-popup @click="exportImage('png')">
            <q-item-section>
              <q-item-label>下载原图 (PNG)</q-item-label>
            </q-item-section>
          </q-item>
          <q-item clickable v-close-popup @click="exportImage('clipboard')">
            <q-item-section>
              <q-item-label>复制缩略图到剪贴板</q-item-label>
            </q-item-section>
          </q-item>
          <q-item clickable v-close-popup @click="exportImage('countdown')">
            <q-item-section>
              <q-item-label>仅下载倒计时素材</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-btn-dropdown>
    </q-card-actions>

    <div>
      <h4 class="q-my-none q-pb-sm">基础</h4>
      <div class="q-col-gutter-sm q-mb-sm">
        <q-file filled v-model="image" label="背景图片" stack-label :model-value="image"
              @update:model-value="refresh"/>
        <q-input filled v-model="texts.top" label="顶部文字" :model-value="texts.top"
               @update:model-value="refresh"/>
        <q-input filled v-model="texts.start" label="起始文字" :model-value="texts.start"
               @update:model-value="refresh"/>
        <q-input filled type="number" v-model="texts.countdown" label="倒计时数字" :model-value="texts.countdown"
               @update:model-value="refresh"/>
        <q-input filled v-model="texts.end" label="结束文字" :model-value="texts.end"
               @update:model-value="refresh"/>
        <q-input filled type="textarea" v-model="texts.sub" label="底部文字" :model-value="texts.sub"
               @update:model-value="refresh"/>
      </div>
    </div>

    <div>
      <h4 class="q-my-none q-pb-sm">高级</h4>
      <div>
        <q-input filled v-model="advanced.textColor" :rules="['anyColor']"
               label="文字颜色" :model-value="advanced.textColor"
               @update:model-value="refresh">
          <template v-slot:append>
            <q-icon name="colorize" class="cursor-pointer">
              <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                <q-color v-model="advanced.textColor" :model-value="advanced.textColor"
                       @change="refresh"/>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <q-input filled v-model="advanced.hintColor" :rules="['anyColor']"
               label="高亮颜色" :model-value="advanced.hintColor"
               @update:model-value="refresh">
          <template v-slot:append>
            <q-icon name="colorize" class="cursor-pointer">
              <q-popup-proxy cover transition-show="scale" transition-hide="scale">
                <q-color v-model="advanced.hintColor" :model-value="advanced.hintColor"
                       @change="refresh"/>
              </q-popup-proxy>
            </q-icon>
          </template>
        </q-input>
        <div>
          <a class="text-subtitle2">标题文字字间距</a>
          <q-slider v-model="advanced.mainSpacing" :min="-250" :max="250" :step="1" label
                    @update:model-value="refresh" :model-value="advanced.mainSpacing"/>
        </div>
        <div>
          <a class="text-subtitle2">底部文字字间距</a>
          <q-slider v-model="advanced.subSpacing" :min="-250" :max="250" :step="1" label
                    @update:model-value="refresh" :model-value="advanced.subSpacing"/>
        </div>
      </div>
    </div>

    <div>
      <h4 class="q-my-none q-pb-sm">鸣谢</h4>
      <q-list>
        <q-item v-for="(item, index) in suggestion" v-bind:key="index" clickable v-ripple>
          <q-item-section>
            <q-item-label>{{ item.title }}</q-item-label>
            <q-item-label caption>{{ item.sub }}</q-item-label>
          </q-item-section>

        </q-item>
      </q-list>
    </div>

  </q-drawer>
</template>

<script>
export default {
  name: "CanvasConfigs",
  data() {
    return {
      drawerRight: true,

      image: null,
      texts: {
        top: "距填写倒计时文案",
        start: "大约还有",
        countdown: 30,
        end: "秒",
        sub: "input countdown contents\nless than 30 seconds"
      },
      advanced: {
        textColor: "#FFFFFF",
        hintColor: "#FF0000",
        mainSpacing: -100,
        subSpacing: 0
      },
      suggestion: [
        {title: "@Rominwolf", sub: "项目制作"},
        {title: "字魂59号-创粗黑", sub: "字体"},
        {title: "Alte DIN 1451 Mittelschrift gepraegt Regular", sub: "字体"},
        {title: "论2023年电影《流浪地球2》中使用的字体（附ttf） - https://www.bilibili.com/read/cv21439547", sub: "文章"},
      ]
    }
  },

  methods: {
    exportImage(type) {
      this.$bus.emit("canvas-export", type);
    },

    refresh() {
      let imageBlob = null;

      if(this.image !== null)
        imageBlob = window.URL.createObjectURL(this.image);

      this.$bus.emit('canvas-refresh', this.texts, imageBlob, this.advanced)
    }
  },

  mounted() {
    this.refresh();
  }
}
</script>
