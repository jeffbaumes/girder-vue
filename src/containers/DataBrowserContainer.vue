<template lang="pug">
div
  slot
    data-browser(ref="view", :model="model", :breadcrumbs="breadcrumbs", :items="items",
        :folders="folders", :loading="fetching", :router-links="routerLinks",
        @uploadComplete="fetch", @folderCreated="fetch")
      slot(v-for="name in viewSlots", :name="name", :slot="name")
</template>

<script>
import rest from '../rest';
import { fetchingContainer, viewSlotWrapper } from '../utils/mixins';
import DataBrowser from '../views/DataBrowser';

export default {
  components: { DataBrowser },
  mixins: [fetchingContainer, viewSlotWrapper],
  props: {
    model: {
      required: true,
      type: Object,
    },
    routerLinks: {
      default: true,
      type: Boolean,
    },
  },
  data: () => ({
    breadcrumbs: [],
    items: [],
    folders: [],
    fetching: false,
  }),
  computed: {
    modelType() {
      return this.model._modelType;
    },
  },
  watch: {
    model() {
      this.fetch();
    },
  },
  methods: {
    fetch() {
      this.$refs.view.showUploader = false;
      this.fetching = true;
      this.items = [];
      this.folders = [];

      let fetchedFolders;
      let fetchedItems = [];
      const requests = [rest.get('/folder', {
        params: {
          parentId: this.model._id,
          parentType: this.modelType,
        },
      }).then(({ data }) => {
        fetchedFolders = data;
      })];

      if (this.modelType === 'folder') {
        const fetchItems = rest.get('/item', {
          params: {
            folderId: this.model._id,
          },
        }).then(({ data }) => {
          fetchedItems = data;
        });

        const fetchRootPath = rest.get(`/folder/${this.model._id}/rootpath`).then(({ data }) => {
          this.breadcrumbs = data.concat([{
            type: this.modelType,
            object: this.model,
          }]);
        });

        requests.push(fetchItems, fetchRootPath);
      } else {
        this.breadcrumbs = [{
          type: this.modelType,
          object: this.model,
        }];
      }

      return Promise.all(requests).finally(() => {
        this.folders = fetchedFolders;
        this.items = fetchedItems;
        this.fetching = false;
      });
    },
  },
};
</script>
