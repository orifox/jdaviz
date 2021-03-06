<template>
  <component :is="stack.container">
    <g-viewer-tab
      v-for="(child, index) in stack.children"
      :stack="child"
      :key="index"
      :data-items="dataItems"
      @resize="$emit('resize')"
      @destroy="$emit('destroy', $event)"
      @data-item-selected="$emit('data-item-selected', $event)"
      @save-figure="$emit('save-figure', $event)"
    ></g-viewer-tab>
    <gl-component
      v-for="(viewer, index) in stack.viewers"
      :key="index"
      title="Viewer"
      :tab-id="viewer.id"
      @resize="$emit('resize')"
      @destroy="$emit('destroy', viewer.id)"
      style="display: flex; flex-flow: column; overflow: hidden"
    >
        <div>
          <v-row dense style="background-color: #205f76">
            <v-col md="auto">
              <v-menu offset-y :close-on-content-click="false">
                <template v-slot:activator="{ on, attrs }">
                    <v-btn
                      text
                      dark
                      elevation="3"
                      v-bind="attrs"
                      v-on="on"
                    >
                      Data
                    </v-btn>
                </template>
                <v-list style="max-height: 500px; width: 350px;" class="overflow-y-auto">
                    <v-checkbox
                      v-for="item in dataItems" :key="item.id" :label="item.name" dense hide-details
                      :input-value="viewer.selected_data_items.includes(item.id)"
                      @change="$emit('data-item-selected', {
                        id: viewer.id,
                        item_id: item.id,
                        checked: $event
                      })"
                      class="pl-4"
                    />
                </v-list>
              </v-menu>
              </v-col>
              <v-spacer></v-spacer>
               <v-col md="auto">
               <v-btn icon @click="viewer.collapse = !viewer.collapse" color="white">
                 <v-icon v-if="viewer.collapse">mdi-hammer-screwdriver</v-icon>
                 <v-icon v-else>mdi-close</v-icon>
               </v-btn>
                <v-menu offset-y :close-on-content-click="false" style="z-index: 10">
                  <template v-slot:activator="{ on }">
                    <v-btn icon color="white" v-on="on">
                      <v-icon >tune</v-icon>
                    </v-btn>
                  </template>

                  <v-tabs v-model="viewer.tab" grow height="36px">
                    <v-tab key="0">Layer</v-tab>
                    <v-tab key="1">Viewer</v-tab>
                  </v-tabs>

                  <v-tabs-items v-model="viewer.tab" style="max-height: 500px; width: 350px;">

                    <v-tab-item key="0" class="overflow-y-auto" style="height: 100%">
                      <v-sheet class="px-4">
                        <jupyter-widget :widget="viewer.layer_options" />
                      </v-sheet>
                    </v-tab-item>

                    <v-tab-item key="1" eager class="overflow-y-auto" style="height: 100%">
                      <v-sheet class="px-4">
                        <jupyter-widget :widget="viewer.viewer_options" />
                      </v-sheet>
                    </v-tab-item>
                  </v-tabs-items>
                </v-menu>
                <v-btn icon color="white">
                  <v-icon >more_horiz</v-icon>
                </v-btn>
              </v-col>
          </v-row>

        </div>

        <v-card tile flat style="flex: 1; margin-top: -2px;">

        <v-toolbar
          dense
          floating
          absolute
          right
          :collapse="viewer.collapse"
          elevation="1"
          :width="viewer.collapse ? '0px' : null"
        >
          <v-toolbar-items>

            <!-- <v-divider vertical></v-divider> -->
            <jupyter-widget :widget="viewer.tools"></jupyter-widget>
            <v-menu offset-y :close-on-content-click="true" style="z-index: 10">
             <template v-slot:activator="{ on }">
              <v-btn icon color="primary" v-on="on">
               <v-icon>mdi-content-save</v-icon>
              </v-btn>
             </template>
             <v-list>
              <v-list-item>
               <v-btn
                color="primary"
                @click="$emit('save-figure', {'id': viewer.id, 'filetype': 'png'})"
               >
                Save as PNG
               </v-btn>
              </v-list-item>
              <v-list-item>
               <v-btn
                color="primary"
                @click="$emit('save-figure', {'id': viewer.id, 'filetype': 'svg'})"
               >
                Save as SVG
               </v-btn>
              </v-list-item>
             </v-list>
            </v-menu>
          </v-toolbar-items>
        </v-toolbar>
        <jupyter-widget :widget="viewer.widget" style="width: 100%; height: 100%" />
      </v-card>
    </gl-component>
  </component>
</template>

<script>
module.exports = {
  name: "g-viewer-tab",
  props: ["stack", "dataItems"],
  created() {
    this.$parent.childMe = () => {
      return this.$children[0];
    };
  },
  methods: {
    computeChildrenPath() {
      return this.$parent.computeChildrenPath();
    }
  },
  computed: {
    parentMe() {
      return this.$parent;
    },
    childMe() {
      return this.$children[0];
    }
  }
};
</script>
