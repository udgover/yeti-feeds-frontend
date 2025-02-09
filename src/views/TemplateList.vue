<template>
  <div class="columns">
    <div class="column is-4">
      <b-table :data="templates" :hoverable="true" :narrowed="true" @click="selectTemplate">
        <template v-slot:default="template">
          <b-table-column field="name" label="Name">
            {{ template.row.name }}
          </b-table-column>
          <b-table-column field="id" label="ID">
            {{ template.row.id }}
          </b-table-column>
        </template>
      </b-table>
    </div>
    <div class="column is-8">
      <b-field label="Name">
        <b-input required ref="templateName" v-model="templateName"></b-input>
      </b-field>
      <b-field label="Template content">
        <b-input required ref="templateContent" v-model="templateContent" type="textarea"></b-input>
      </b-field>
      <b-field grouped>
        <p class="control" v-if="templateId">
          <button class="button is-primary" @click="updateTemplate">Save changes</button>
        </p>
        <p class="control" v-if="templateId">
          <button class="button is-danger" @click="confirmDeleteTemplate">Delete template</button>
        </p>
        <p class="control" v-if="!templateId">
          <button class="button is-primary" @click="newTemplate">Add new template</button>
        </p>
        <p class="control">
          <button class="button is-light" @click="deselectTemplate">Clear form</button>
        </p>
      </b-field>
      The rendering engine receives a <code>data</code> variable containing all selected observables. <br />Example
      template body:

      <span v-pre>
        <pre>
&lt;arbitrary&gt;
{% for obs in data %}{{ obs.value }}
{% endfor %}
&lt;/arbitrary&gt;</pre
        >
      </span>
      Refer to the Yeti documentation for tips on how to write good templates.
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "TemplateList",
  data() {
    return {
      templates: [],
      templateName: null,
      templateContent: null,
      templateId: null
    };
  },
  mounted() {
    this.listExportTemplates();
  },
  methods: {
    listExportTemplates() {
      axios
        .post("/api/v2/templates/search", { query: { name: "" } })
        .then(response => {
          this.templates = response.data.templates;
        })
        .catch(error => {
          console.log(error);
        })
        .finally(() => {});
    },
    selectTemplate(row) {
      this.templateName = row.name;
      this.templateContent = row.template;
      this.templateId = row.id;
    },
    deselectTemplate() {
      this.templateName = this.templateContent = this.templateId = null;
    },
    newTemplate() {
      var valid = this.$refs.templateName.checkHtml5Validity();
      valid &= this.$refs.templateContent.checkHtml5Validity();
      if (!valid) {
        return;
      }
      var params = {
        template: {
          name: this.templateName,
          template: this.templateContent
        }
      };
      axios
        .post(`/api/v2/templates/`, params)
        .then(() => {
          this.deselectTemplate();
          this.listExportTemplates();
        })
        .catch(error => {
          console.log(error);
        })
        .finally(() => {});
    },
    updateTemplate() {
      var valid = this.$refs.templateName.checkHtml5Validity();
      valid &= this.$refs.templateContent.checkHtml5Validity();
      if (!valid) {
        return;
      }
      var params = {
        template: {
          id: this.templateId,
          name: this.templateName,
          template: this.templateContent
        }
      };
      axios
        .patch(`/api/v2/templates/${this.templateId}`, params)
        .then(() => {
          this.deselectTemplate();
          this.listExportTemplates();
        })
        .catch(error => {
          console.log(error);
        })
        .finally(() => {});
    },
    confirmDeleteTemplate() {
      this.$buefy.dialog.confirm({
        title: "Delete template",
        message: `You're about to <b>delete</b> Template <code>${this.templateName}</code>. Proceed?`,
        confirmText: "Delete template",
        type: "is-danger",
        hasIcon: true,
        onConfirm: this.deleteTemplate
      });
    },
    deleteTemplate() {
      axios
        .delete(`/api/v2/templates/${this.templateId}`)
        .then(() => {
          this.deselectTemplate();
        })
        .catch(error => {
          console.log(error);
        })
        .finally(() => {
          this.listExportTemplates();
        });
    }
  }
};
</script>
