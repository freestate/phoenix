<template>
  <v-list two-line>
    <v-list-tile>
      <v-list-tile-action>
        <v-checkbox
          :input-value="all"
          primary
          hide-details
          @click.native="toggleAll"
        ></v-checkbox>
      </v-list-tile-action>
      <v-list-tile-action>
      </v-list-tile-action>
      <v-list-tile-content>
        <v-list-tile-title v-translate>Name</v-list-tile-title>
        <v-list-tile-sub-title class="text--primary" v-translate>Size</v-list-tile-sub-title>
        <v-list-tile-sub-title v-translate>Modification Time</v-list-tile-sub-title>
      </v-list-tile-content>
    </v-list-tile>
    <v-divider></v-divider>
    <template v-for="(item, index) in fileData">
      <v-list-tile
        :key="item.path"
        avatar
        ripple
      >
        <v-list-tile-action>
          <v-checkbox
            @change="$emit('toggle', item)"
            :input-value="selectedFiles.indexOf(item) >= 0"
            primary
            hide-details></v-checkbox>
        </v-list-tile-action>
        <v-list-tile-action>
          <v-checkbox
            @change="toggleFileFavorite(item)"
            :input-value="item.starred"
            primary
            hide-details
            color="yellow"
            on-icon="star" off-icon="star_border" large></v-checkbox>
        </v-list-tile-action>
        <v-list-tile-content
          @click="item.extension === false ? navigateTo('files-list', item.path) : openFileActionBar(item)"
          style="cursor: pointer"
        >
          <v-list-tile-title>{{ item.name }}</v-list-tile-title>
          <v-list-tile-sub-title class="text--primary">{{ item.size | fileSize }}</v-list-tile-sub-title>
          <v-list-tile-sub-title>{{ item.mdate | formDateFromNow }}</v-list-tile-sub-title>
        </v-list-tile-content>

        <v-list-tile-action>
          <v-btn icon>
            <v-icon>edit</v-icon>
          </v-btn>
        </v-list-tile-action>
        <v-list-tile-action>
          <v-btn icon>
            <v-icon>file_download</v-icon>
          </v-btn>
        </v-list-tile-action>
        <v-list-tile-action>
          <v-btn icon>
            <v-icon>delete</v-icon>
          </v-btn>
        </v-list-tile-action>

      </v-list-tile>
      <v-divider
        v-if="index + 1 < fileData.length"
        :key="index"
      ></v-divider>
    </template>
  </v-list>
</template>
<script>

import { includes } from 'lodash'
import { mapGetters, mapActions } from 'vuex'

import Mixins from '../mixins'

export default {
  mixins: [
    Mixins
  ],
  name: 'FileList',
  props: ['fileData', 'starsEnabled', 'checkboxEnabled', 'dateEnabled'],
  data: () => ({
    columnsDisabled: {
      favorite: false,
      fileSelect: false,
      data: false
    },
    headers: [
      { text: 'Name', value: 'name' },
      { text: 'Size', value: 'size' },
      { text: 'Date', value: 'date' }
    ],
    pagination: {
      sortBy: 'name'
    }
  }),
  methods: {
    ...mapActions('Files', ['markFavorite', 'resetFileSelection', 'addFileSelection', 'removeFileSelection']),
    ...mapActions(['openFile']),

    toggleAll () {
      if (this.selectedFiles.length) {
        this.resetFileSelection()
      } else {
        let selectedFiles = this.fileData.slice()
        for (let item of selectedFiles) {
          if (!includes(this.selectedFiles, item)) {
            this.addFileSelection(item)
          }
        }
      }
    },
    toggleFileFavorite (item) {
      this.markFavorite({
        client: this.$client,
        file: item
      })
    },
    disableColumn (column) {
      if (column === 'stars') {
        this.columnsDisabled.favorite = this.starsEnabled !== false
        return this.columnsDisabled.favorite
      } else if (column === 'checkbox') {
        this.columnsDisabled.fileSelect = this.checkboxEnabled !== false
        return this.columnsDisabled.fileSelect
      }
      return true
    },
    changeSort (column) {
      if (this.pagination.sortBy === column) {
        this.pagination.descending = !this.pagination.descending
      } else {
        this.pagination.sortBy = column
        this.pagination.descending = false
      }
    },
    openFileActionBar (file) {
      this.$emit('FileAction', file)
    }
  },
  computed: {
    ...mapGetters('Files', ['selectedFiles']),
    all() {
      return this.selectedFiles.length === this.fileData.length
    }
  }
}

</script>
