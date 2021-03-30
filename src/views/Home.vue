<template>
  <a-card title="Тестовое задание" style="text-align: left">
    <div slot="extra">
      <a-input-search
        placeholder="Поиск сделок"
        v-model="message"
        :loading="loading"
      />
    </div>
    <a-table
      bordered
      :pagination="false"
      :columns="columns"
      :data-source="leads"
      :customRow="customClick"
      :loading="loading"
      :rowKey="(record) => record.id"
    >
      <template slot="name" slot-scope="text, record, index">
        <div style="font-weight: 600">
          {{ text }}
        </div>
      </template>
      <div slot="status" slot-scope="text">
        <a-tag style="color: #000" :color="text.color">{{ text.name }}</a-tag>
      </div>
      <div slot="user" slot-scope="text">{{ text[0] }} {{ text }}</div>
      <div slot="price" slot-scope="text">
        {{ text.toLocaleString() }}
      </div>
      <div slot="created_at" slot-scope="text">
        {{ fotmatDate(text) }}
      </div>
    </a-table>
  </a-card>
</template>

<script lang="ts">
import { Component, Vue, Watch } from 'vue-property-decorator';
import axios from 'axios';
import moment from 'moment';
import { Debounce } from 'vue-debounce-decorator';

require('moment/locale/ru');
moment.locale('ru');

@Component({})
export default class Home extends Vue {
  leads: unknown[] = [];
  message = '';
  loading = false;
  customClick = (record: any) => ({
    on: {
      click: () => {
        this.$router.push({ path: `/contact/${record.id}` });
      },
    },
  });
  columns = [
    {
      title: 'Название',
      dataIndex: 'name',
      key: 'name',
      scopedSlots: { customRender: 'name' },
    },
    {
      title: 'Статус',
      dataIndex: 'pipeline',
      key: 'pipeline',
      scopedSlots: { customRender: 'status' },
    },
    {
      title: 'Ответственный',
      dataIndex: 'user.name',
      key: 'user',
      scopedSlots: { customRender: 'user' },
    },
    {
      title: 'Дата создания',
      dataIndex: 'created_at',
      key: 'created_at',
      scopedSlots: { customRender: 'created_at' },
    },
    {
      title: 'Бюджет',
      dataIndex: 'price',
      key: 'price',
      scopedSlots: { customRender: 'price' },
    },
  ];

  @Watch('message')
  @Debounce(500)
  handleChange(val: string) {
    this.getLeads({ name: val });
  }

  mounted(): void {
    this.getLeads();
    if (this.$route.query.name) {
      console.log(this.$route.query.name);
      this.getLeads({ name: this.$route.query.name });
    }
  }

  fotmatDate(val: number): string {
    return moment.unix(val).format('D MMMM YYYY');
  }

  getLeads(params?: object) {
    this.loading = true;
    axios
      .get('http://localhost:3000/leads', {
        params,
      })
      .then(({ data }) => {
        this.leads = [...data];

        if (params) {
          this.$router.push({
            path: '/search',
            query: { name: this.message },
          });
        } else {
          return;
        }
      })

      .finally(() => (this.loading = false));
  }
}
</script>

<style>
tbody tr {
  cursor: pointer;
}
</style>
