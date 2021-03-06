<template>
  <div class="mt-1">
    <vue-tags-input
      v-model="tag"
      :tags="tags"
      @before-deleting-tag="onDeleteTagBookmark"
      @tags-changed="(newTags) => (tags = newTags)"
      @before-adding-tag="onAddTagBookmark"
      placeholder="태그 추가"
    />
  </div>
</template>

<script>
import VueTagsInput from '@johmun/vue-tags-input';
import { mapMutations, mapActions, mapGetters } from 'vuex';
import {
  ADD_TAG_ON_BOOKMARK,
  CREATE_TAG,
  DELETE_TAG_ON_BOOKMARK,
  FETCH_BOOKMARK_WITH_TAGS,
} from '@/store/share/actionsType.js';
import { BOOKMARK_ID, TAGS_WITH_BOOKMARK, GET_TAG_ID_BY_NAME } from '@/store/share/gettersType.js';
import { SHOW_SNACKBAR } from '@/store/share/mutationsType.js';
import { SNACKBAR_MESSAGES } from '@/utils/constants.js';

export default {
  name: 'TagInput',
  components: {
    VueTagsInput,
  },
  data() {
    return {
      tag: '',
      tags: [],
      tagCreateRequest: {
        name: '',
      },
    };
  },
  computed: {
    ...mapGetters([BOOKMARK_ID, GET_TAG_ID_BY_NAME, TAGS_WITH_BOOKMARK]),
  },
  watch: {
    async [BOOKMARK_ID]() {
      if (this[BOOKMARK_ID]) {
        try {
          await this[FETCH_BOOKMARK_WITH_TAGS](this[BOOKMARK_ID]);
          this.initTags();
        } catch (e) {
          this[SHOW_SNACKBAR](SNACKBAR_MESSAGES.TAG_BOOKMARK.FETCH.FAIL);
        }
      }
    },
  },
  methods: {
    ...mapMutations([SHOW_SNACKBAR]),
    ...mapActions([ADD_TAG_ON_BOOKMARK, CREATE_TAG, DELETE_TAG_ON_BOOKMARK, FETCH_BOOKMARK_WITH_TAGS]),
    async onAddTagBookmark(data) {
      this.tagCreateRequest.name = data.tag.text;
      try {
        const createdTagId = await this[CREATE_TAG](this.tagCreateRequest);
        await this[ADD_TAG_ON_BOOKMARK]({
          bookmarkId: this[BOOKMARK_ID],
          tagId: createdTagId,
        });
        await this[FETCH_BOOKMARK_WITH_TAGS](this[BOOKMARK_ID]);
        data.addTag();
      } catch (e) {
        this[SHOW_SNACKBAR](SNACKBAR_MESSAGES.TAG_BOOKMARK.ADD.FAIL);
      }
    },
    async onDeleteTagBookmark(data) {
      const targetTagName = data.tag.text;
      const targetTagId = this[GET_TAG_ID_BY_NAME](targetTagName);
      try {
        await this[DELETE_TAG_ON_BOOKMARK]({
          bookmarkId: this[BOOKMARK_ID],
          tagId: targetTagId,
        });
        await this[FETCH_BOOKMARK_WITH_TAGS](this[BOOKMARK_ID]);
        data.deleteTag();
      } catch (e) {
        this[SHOW_SNACKBAR](SNACKBAR_MESSAGES.TAG_BOOKMARK.DELETE.FAIL);
      }
    },
    initTags() {
      this.tags = this[TAGS_WITH_BOOKMARK].map((tag) => this.mapTag(tag));
    },
    mapTag(tagValue) {
      return {
        text: tagValue.name,
        tiClasses: ['ti-valid'],
      };
    },
  },
};
</script>
