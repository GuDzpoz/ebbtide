<!--
    Ebbtide - A frontend for lotide
    Copyright (C) 2020  gudzpoz

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU Affero General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU Affero General Public License for more details.

    You should have received a copy of the GNU Affero General Public License
    along with this program.  If not, see <https://www.gnu.org/licenses/>.
-->

<template>
<div>
  <span class="caption">by <router-link :to="getAuthorLink(item.author.id)">{{ item.author.username }}</router-link> on {{ time(item.created) }}</span>
  <div v-if="item.content_html" class="reply" v-html="item.content_html"></div>
  <div v-else-if="item.content_text" class="reply">{{ item.content_text }}</div>
  <w-button @click="like" bg-color="white">
    <w-icon v-if="liked" color="warning">mdi mdi-star</w-icon>
    <w-icon v-else>mdi mdi-star-outline</w-icon>
    {{ number }}
  </w-button>
  <w-button @click="replying = !replying" icon="mdi mdi-reply-outline" text lg color="info">Reply</w-button>
  <w-button @click="this.$emit('reply', { item: item, reply: reply })" v-if="replying" color="info" class="ml3">Submit</w-button>
  <w-textarea v-if="replying" v-model:model-value="reply">Comment (Markdown supported)</w-textarea>
  <div class="ml6 column">
    <Comment column class="mt2" v-for="i in replies" :key="i.created" :item="i" @reply="passon"></Comment>
    <w-button v-if="item.has_replies && !item.replies && !loadedReplies && !loadingReplies" @click="loadReplies">Load replies</w-button>
    <w-spinner v-if="loadingReplies" />
  </div>
</div>
</template>

<script>
import moment from 'moment'

export default {
  name: 'Comment',
  emits: ['reply'],
  props: {
    item: Object
  },
  methods: {
    like () {
      if(this.liked) {
        this.$lotide.unlikeComment(this.item.id)
        --this.number
      } else {
        this.$lotide.likeComment(this.item.id)
        ++this.number
      }
      this.liked = !this.liked
    },
    time (utc) {
      return moment.utc(utc).fromNow()
    },
    getAuthorLink (id) {
      return '/main/user/' + id
    },
    passon (item) {
      this.$emit('reply', item)
    },
    loadReplies () {
      this.loadingReplies = true
      this.$lotide.getComment(this.item.id).then((json) => {
        this.loadingReplies = false
        this.loadedReplies = json.replies
      })
    }
  },
  computed: {
    replies () {
      if(this.item.replies) {
        return this.item.replies
      } else {
        return this.loadedReplies
      }
    }
  },
  data () {
    return {
      replying: false,
      liked: false,
      number: null,
      loadingReplies: false,
      loadedReplies: null,
      reply: ''
    }
  },
  created () {
    if(this.item.your_vote) {
      this.liked = true
    } else {
      this.liked = false
    }
    this.number = this.item.score
  }
}
</script>
