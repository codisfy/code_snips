<template>
  <div>
    <form
      class="form-horizontal"
      :action="action"
      :method="method"
      @submit.prevent="onSubmit($event.target.action)"
      v-if="!pageLoading"
    >
      <div class="card-body">
        <template v-for="(categoryData, categoryGroup) in categories">
          <p class="f-w-600 font-size-20 m-t-20" :key="'p'+categoryGroup">{{categoryGroup}}</p>
          <div class="form-group row pl-3" :key="'div'+categoryGroup">
            <div class="col-12">
              <nested-checkboxes
                :key="'parent' + categoryGroup"
                :category-data="getTree(categoryData, categoryData, [])"
                :root-data="categoryData"
                @checkbox-updated="handleUpdatedEvent"
                :form-data="formData.categories"
              ></nested-checkboxes>
            </div>
          </div>
        </template>
      </div>
      <div class="card-footer bg-light">
        <div class="form-actions">
          <div class="row">
            <div class="col-md-12">
              <div class="row">
                <div class="offset-sm-3 col-md-5">
                  <button class="btn btn-primary btn-rounded" type="submit">Submit</button>
                  <a
                    class="btn btn-secondary clear-form btn-rounded btn-outline"
                    :href="cancelUrl"
                  >Cancel</a>
                </div>
              </div>
              <div
                v-if="error"
                class="alert alert-danger alert-dismissible fade show"
                role="alert"
              >{{error}}</div>
            </div>
          </div>
        </div>
      </div>
    </form>
    <div v-if="pageLoading">
      <loading></loading>
    </div>
  </div>
</template>

<script>
import mixin from "../../common/mixin";
import NestedCheckboxes from "./NestedCheckboxes";
export default {
  name: "SyncCategories",
  mixins: [mixin],
  components: {
    NestedCheckboxes: NestedCheckboxes
  },
  data: function() {
    return {
      action:
        this.appUrl + "/api/v1/quickbooks/sync-categories/" + this.company.id,
      method: "POST",
      categories: {},
      rendered: [], // this will keep track of checkboxes already shown
      formData: {
        categories: []
      }
    };
  },
  methods: {
    successCallback(response) {
      this.nextPage(response);
    },
    setup() {
      this.request(
        {
          url: this.action,
          method: "GET"
        },
        this.getCategories
      );
    },
    getCategories(response) {
      this.categories = response.body;
    },
    nextPage(response) {
    },
    getTree(allCategories, subCategories, attached) {
      var tree = [];
      var counter = 0;
      subCategories.forEach(ex => {
        if (!attached.includes(ex.Id)) {
          attached.push(ex.Id);
          tree[counter] = ex;
          var children = this.getChildren(ex.Id, allCategories);
          if (!children || !children.length) {
            tree[counter]["children"] = children;
          } else {
            tree[counter]["children"] = this.getTree(
              allCategories,
              children,
              attached
            );
          }
          counter++;
        }
      });
      return tree;
    },
    getChildren(id, categoryData) {
      return _.filter(categoryData, function(o) {
        return o.ParentRef == id;
      });
    },
    handleUpdatedEvent(data) {
      if (data.checked) {
        this.$set(
          this.formData,
          "categories",
          _.union(this.formData.categories, data.value)
        );
      } else {
        this.$set(
          this.formData,
          "categories",
          _.difference(this.formData.categories, data.value)
        );
      }
    }
  }
};
</script>