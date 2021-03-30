<template>
  <a-card :title="contact.name" style="text-align: left; font-size: 16px">
    <a-tag
      v-if="contact.pipeline"
      slot="extra"
      style="color: #000"
      :color="contact.pipeline.color"
      >{{ contact.pipeline.name }}</a-tag
    >
    <a-table
      bordered
      :pagination="false"
      :rowKey="(record) => record.id"
      :loading="loading"
      :columns="columns"
      :data-source="contact.contacts"
    >
      <div slot="first_name" slot-scope="text, record" class="cs">
        <!--        {{ record }}-->
        <dl>
          <dt>Имя:</dt>
          <dd>{{ record.first_name }} {{ record.last_name }}</dd>
        </dl>

        <dl v-for="f in record.custom_fields_values" :key="f.id">
          <dt>{{ f.field_name }}:</dt>
          <span v-for="v in f.values" :key="v.id">
            <dd>{{ v.value }}</dd>
          </span>
        </dl>
      </div>
    </a-table>
  </a-card>
</template>

<script lang="ts">
import Vue from 'vue';
import Component from 'vue-class-component';
import axios from 'axios';

@Component({})
export default class Contacts extends Vue {
  contact = {};
  loading = false;
  columns = [
    {
      title: 'Контакт',
      dataIndex: 'first_name',
      key: 'first_name',
      scopedSlots: { customRender: 'first_name' },
    },
  ];

  mounted(): void {
    this.loading = true;
    const id = parseInt(this.$route.params.id);
    axios
      .get(`http://localhost:3000/leads/${id}`)
      .then(({ data }) => (this.contact = { ...data }))
      .finally(() => (this.loading = false));
  }

  goBack(): void {
    this.$router.go(-1);
  }
}
</script>

<style scoped>
.ant-list-item {
  flex-direction: column;
  text-align: left;
  align-items: start;
}
.ant-card-head-title {
  text-align: left;
  font-size: 16px !important;
}
dd,
.ant-table-column-title {
  font-size: 16px;
  font-weight: bold;
}
tr.ant-table-row {
  cursor: default !important;
}
</style>
