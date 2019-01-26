<template>
  <div class="row p-0" :class="depth ? 'ml-3' : ''">
    <template v-for="(qbCategory) in categoryData">
      <div
        class="pl-2 p-1"
        :class="depth ? 'col-12' : 'col-4 border border-light'"
        :key="depth + qbCategory.Id"
      >
        <div class="custom-control custom-checkbox checkbox-primary form-check">
          <input
            type="checkbox"
            class="custom-control-input"
            :id="'qbCategory' + qbCategory.Id"
            :value="qbCategory.Id"
            @input="updatedValue"
            v-model="selectedData"
          >
          <label
            class="custom-control-label"
            :for="'qbCategory' + qbCategory.Id"
          >{{ qbCategory.Name}}</label>
        </div>
        <nested-checkboxes
          :key="'child' + qbCategory.Id + depth"
          :category-data="qbCategory.children"
          :depth="depth+1"
          :form-data="formData"
          :root-data="rootData"
          @checkbox-updated="handleUpdatedEvent"
        ></nested-checkboxes>
      </div>
    </template>
  </div>
</template>
<script>
export default {
  name: "NestedCheckboxes",
  props: {
    categoryData: {
      required: false
    },
    rootData: {
      required: false
    },
    depth: {
      default: 0
    },
    formData: {
      default: function() {
        return [];
      }
    }
  },
  data() {
    return {
      selectedData: []
    };
  },
  updated() {
    this.selectedData = this.formData;
  },
  methods: {
    updatedValue(e) {
      let values = [e.target.value];
      let children = this.getChildren(this.rootData, values);
      if (children && children.length) {
            values = _.union(values, children);
      }
      let eventData = {
        value: values,
        checked: e.target.checked
      };
      this.$emit("checkbox-updated", eventData);
    },
    handleUpdatedEvent(data) {
      if (data.checked) {
        this.selectedData = _.union(this.selectedData, data.value);
      } else {
        this.selectedData = _.difference(this.selectedData, data.value);
      }
      this.$emit("checkbox-updated", data);
    }, 
    getChildren(categoryData, children = [], processed = []) {
        let childrenToProcess = _.difference(children, processed);
        childrenToProcess.forEach((child) => {
            processed.push(child);
            let subChildren =  _.map(_.filter(categoryData, function(o) {
                return o.ParentRef == child;
            }), 'Id');
            if (subChildren.length) {
                children = _.union(children, this.getChildren(categoryData, subChildren, processed));
            } 
        });
        return processed;
    }, 
  }
};
</script>