<template>
  <smart-widget title="词云" :loading="loading" :simple="simple">
    <WordCloud :word_freq="word_freq" ref="wordcloud" />
  </smart-widget>
</template>

<script>
/* eslint-disable */
import _ from "lodash";
import WordCloud from "./WordCloud";

//import "../utils/require-jieba-js";
import jieba from "../utils/main";

import stopwords from "raw-loader!../assets/edited_baidu_stopwords.txt";

export default {
  components: {
    WordCloud
  },
  props: {
    plot: {
      type: String
    },
    userDict: {
      type: Array
    },
    simple: {
      type: Boolean
    }
  },

  data: () => ({
    word_freq: undefined,
    loading: true,
    stopwords: new Set(
      stopwords
        .trim()
        .split("\n")
        .map(t => t.trim())
    )
  }),

  methods: {
    resizeEvent: function() {
      this.resizeFunc();
    },
    refreshWordFreq: function() {
      if (this.plot == undefined) {
        this.loading = true;
        word_freq = {};
        return;
      }
      this.loading = true;
      jieba.jieba_cut(this.plot, res => {
        res = _.filter(
          res,
          word =>
            !this.stopwords.has(word) &&
            typeof word != "function" &&
            !word.startsWith("function")
        );
        var count = _.countBy(res);
        this.word_freq = count;
        this.loading = false;
      });
    }
  },
  mounted: function() {
    //防抖动，降低重绘开销，500ms
    this.resizeFunc = _.debounce(this.$refs.wordcloud.resize, 500);
    this.resizeFunc(); //绘制完成后修改一下尺寸
    this.$parent.$on("resized", this.resizeEvent); //接收外层resize事件
    this.$parent.$on("container-resized", this.resizeEvent);
  },
  watch: {
    plot: function() {
      this.refreshWordFreq();
    },
    userDict: function() {
      jieba.build_dict(this.userDict);
      this.refreshWordFreq();
    }
  }
};
</script>