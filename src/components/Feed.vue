<template>
  <v-card class="mx-auto" max-width="100%" elevation="15" tile>
    <v-list v-if="feeds.length>0" rounded>
      <v-subheader>FEED</v-subheader>
      <template v-for="feed in feeds">
        <v-card v-bind:key="feed.source">
          <v-row>
            <v-col class="sourceLogo" cols="2" offset-xs2>
              <v-avatar size="64">
                <v-img :src="feed.sourceLogo"></v-img>
              </v-avatar>
            </v-col>
            <v-col cols="10">
              <v-card-title
                class="sourceTitle"
                v-resize-text="{ratio:1.5, minFontSize: '16px', maxFontSize: '30px', delay: 100}"
              >{{feed.source}}</v-card-title>
            </v-col>
          </v-row>
          <v-container>
            <v-card v-for="post in feed.sourcePosts" v-bind:key="post.tile">
              <v-card-title
                v-resize-text="{ratio:2, minFontSize: '16px', maxFontSize: '18px', delay: 100}"
              >
                <a class="sourcePostTitle" @click="openLink(post.link)">{{post.title}}</a>
              </v-card-title>
              <v-card-text>{{post.description}}</v-card-text>
            </v-card>
          </v-container>
        </v-card>
      </template>
    </v-list>
    <v-container class="progressCircular" fluid>
      <v-progress-circular v-if="!feeds ||!feeds.length>0 " indeterminate color="primary"></v-progress-circular>
    </v-container>
  </v-card>
</template>

<script>
import axios from "axios";
import { parseString } from "xml2js";
import ResizeText from "vue-resize-text";

export default {
  name: "Feed",
  directives: {
    ResizeText
  },
  data: () => ({
    rssLinks: [
      "https://rss.app/feeds/EbP5zHQItWPvUD71.xml",
      "https://rss.app/feeds/ltIxcEDrdPrCw7ry.xml"
    ],
    feeds: []
  }),
  mounted() {
    this.refreshFeed();
    this.countDown();
  },
  methods: {
    async requireFeed(URL) {
      try {
        await axios.get(URL).then(response => {
          parseString(response.data, (err, res2) => {
            this.beautifyData(res2);
          });
        });
        // console.log(data);
        // this.butefyData(data)
      } catch (ex) {
        console.log(ex);
      }
    },
    refreshFeed() {
      this.feeds = [];
      this.rssLinks.forEach(el => {
        this.requireFeed(el);
      });
    },
    beautifyData(data) {
      this.feeds.push({
        source: data.rss.channel[0].title[0],
        sourceLogo: data.rss.channel[0].image[0].url[0],
        sourcePosts: this.filterList("covid", data.rss.channel[0].item).map(
          el => {
            return {
              title: el.title[0],
              description: el.description[0]
                .toString()
                .match(/<div>[^<|>][\s\S]*?<\/div>/)[0]
                .replace(/<div>|<\/div>/g, ""),
              link: el.link[0],
              pubDate: el.pubDate[0]
            };
          }
        )
      });
    },
    //https://www.peterbe.com/plog/a-darn-good-search-filter-function-in-javascript
    filterList(q, list) {
      function escapeRegExp(s) {
        return s.replace(/[-/\\^$*+?.()|[\]{}]/g, "\\$&");
      }
      const words = q
        .split(/\s+/g)
        .map(s => s.trim())
        .filter(s => !!s);
      const hasTrailingSpace = q.endsWith(" ");
      const searchRegex = new RegExp(
        words
          .map((word, i) => {
            // console.log("word", word);
            if (i + 1 === words.length && !hasTrailingSpace) {
              // The last word - ok with the word being "startswith"-like
              return `(?=.*\\b${escapeRegExp(word)})`;
            } else {
              // Not the last word - expect the whole word exactly
              return `(?=.*\\b${escapeRegExp(word)}\\b)`;
            }
          })
          .join("") + ".+",
        "gi"
      );
      return list.filter(item => {
        return (
          searchRegex.test(item.title) || searchRegex.test(item.description)
        );
      });
    },
    openLink(url){
        window.open(url)
    },
    countDown() {
      setInterval(() => {
        // console.log("refresh");
        this.refreshFeed();
      }, 15 * 60000);
    }
  }
};
</script>

<style scoped>
.sourceLogo {
  text-align: center;
  align-self: center;
}
.sourceTitle {
  align-self: center;
  font-family: "Ubuntu", sans-serif;
}
.sourcePostTitle {
  color: black;
}
.sourcePostTitle:hover {
  color: #1565c0;
}
.v-card {
  margin: 5px;
}
.progressCircular {
  margin: 5px;
  text-align: center;
  align-self: center;
}
</style>