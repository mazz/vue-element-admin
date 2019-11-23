<template>
  <div class="app-container">
      <el-dropdown trigger="click" @command="handleSetChannel">
    <div>
      <el-col :span="4" class="text-center">
        <router-link class="pan-btn green-btn" to="/table/complex-table">
          Select Channel
        </router-link>
      </el-col>
    </div>
    <el-dropdown-menu slot="dropdown">
      <el-dropdown-item  v-for="item in channels" :key="item.uuid" :label="item.basename+'('+item.uuid+')'" :value="item.uuid"  :command="item.uuid">
        {{
          item.basename }}
      </el-dropdown-item>
    </el-dropdown-menu>
  </el-dropdown>

    <div class="filter-container">
      <!-- <el-select v-model="listQuery.channel" placeholder="Channel" clearable class="filter-item" style="width: 130px"> -->
        <!-- <el-option v-for="item in channels" :key="item.uuid" :label="item.basename+'('+item.uuid+')'" :value="item.uuid" /> -->
      <!-- </el-select> -->
      <!-- <el-input v-model="listQuery.title" placeholder="Localized Name" style="width: 200px;" class="filter-item" @keyup.enter.native="handleFilter" /> -->
      <!-- <el-select v-model="listQuery.type" placeholder="Media Category" clearable class="filter-item" style="width: 130px"> -->
        <!-- <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name+'('+item.key+')'" :value="item.key" /> -->
      <!-- </el-select> -->
      <!-- <el-select v-model="listQuery.sort" style="width: 140px" class="filter-item" @change="handleFilter"> -->
        <!-- <el-option v-for="item in sortOptions" :key="item.key" :label="item.label" :value="item.key" /> -->
      <!-- </el-select> -->
      <!-- <el-button v-waves class="filter-item" type="primary" icon="el-icon-search" @click="handleFilter"> -->
        <!-- Search -->
      <!-- </el-button> -->
      <el-button class="filter-item" style="margin-left: 10px;" type="primary" icon="el-icon-edit" @click="handleCreate">
        Add
      </el-button>
      <!-- <el-button v-waves :loading="downloadLoading" class="filter-item" type="primary" icon="el-icon-download" @click="handleDownload"> -->
        <!-- Export -->
      <!-- </el-button> -->
    </div>

    <el-table
      :key="tableKey"
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      style="width: 100%;"
      @sort-change="sortChange"
    >
      <el-table-column label="Ordinal" prop="id" sortable="custom" align="center" width="80" :class-name="getSortClass('ordinal')">
        <template slot-scope="{row}">
          <span>{{ row.ordinal }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Updated At" width="150px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.updated_at | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Localized Name" min-width="200px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.localizedname }}</span>
          <el-tag>{{ row.media_category | typeFilter }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="Language ID" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.language_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Hash ID" width="110px" align="center">
        <template slot-scope="{row}">
          <span>{{ row.hash_id }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Small Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.small_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Med Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.med_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Large Thumbnail Path" min-width="150px">
        <template slot-scope="{row}">
          <span class="link-type" @click="handleUpdate(row)">{{ row.large_thumbnail_path }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Actions" align="center" width="230" class-name="small-padding fixed-width">
        <template slot-scope="{row}">
          <el-button type="primary" size="mini" @click="handleUpdate(row)">
            Edit
          </el-button>
          <!-- <el-button v-if="row.status!='deleted'" size="mini" type="danger" @click="handleModifyStatus(row,'deleted')"> -->
            <!-- Delete -->
          <!-- </el-button> -->
        </template>
      </el-table-column>
    </el-table>

    <!-- <pagination v-show="total>0" :total="total" :page.sync="listQuery.page" :limit.sync="listQuery.limit" @pagination="getList" /> -->

    <el-dialog :title="textMap[dialogStatus]" :visible.sync="dialogFormVisible">
      <el-form ref="dataForm" :rules="rules" :model="temp" label-position="left" label-width="70px" style="width: 400px; margin-left:50px;">
        <el-form-item label="Channel" prop="channel">
          <el-select v-model="temp.channel_uuid" class="filter-item" placeholder="Please select">
        <el-option v-for="item in channels" :key="item.uuid" :label="item.basename+'('+item.uuid+')'" :value="item.uuid" />
          </el-select>
        </el-form-item>
        <el-form-item label="Ordinal" prop="ordinal">
          <el-input v-model="temp.ordinal" />
        </el-form-item>
        <el-form-item label="Media Category" prop="media_category">
          <el-select v-model="temp.media_category" class="filter-item" placeholder="Please select">
            <el-option v-for="item in calendarTypeOptions" :key="item.key" :label="item.display_name" :value="item.key" />
          </el-select>
        </el-form-item>
        <el-form-item label="Date" prop="updated_at">
          <el-date-picker v-model="temp.updated_at" type="datetime" placeholder="Please pick a date" />
        </el-form-item>
        <el-form-item label="Localized Name" prop="localizedname">
          <el-input v-model="temp.localizedname" />
        </el-form-item>
        <el-form-item label="Language ID" prop="language_id">
          <el-input v-model="temp.language_id" />
        </el-form-item>
        <el-form-item label="Small Thumbnail Path" prop="small_thumbnail_path">
          <el-input v-model="temp.small_thumbnail_path" />
        </el-form-item>
        <el-form-item label="Medium Thumbnail Path" prop="med_thumbnail_path">
          <el-input v-model="temp.med_thumbnail_path" />
        </el-form-item>
        <el-form-item label="Large Thumbnail Path" prop="large_thumbnail_path">
          <el-input v-model="temp.large_thumbnail_path" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">
          Cancel
        </el-button>
        <el-button type="primary" @click="dialogStatus==='create'?createData():updateData()">
          Confirm
        </el-button>
      </div>
    </el-dialog>

    <el-dialog :visible.sync="dialogPvVisible" title="Reading statistics">
      <el-table :data="pvData" border fit highlight-current-row style="width: 100%">
        <el-table-column prop="key" label="Channel" />
        <el-table-column prop="pv" label="Pv" />
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogPvVisible = false">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import axios from 'axios'
import { fetchList, fetchPv, createArticle, updateArticle } from '@/api/article'
import waves from '@/directive/waves' // waves directive
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination' // secondary package based on el-pagination

const channelOptions = [
  { key: 'bible-uuid', display_name: 'Bible Channel' },
  { key: 'gospel-uuid', display_name: 'Gospel Channel' },
  { key: 'preaching-uuid', display_name: 'Preaching Channel' },
  { key: 'musid-uuid', display_name: 'Music Channel' },
  { key: 'movies-uuid', display_name: 'Movies Channel' }
]

const channelOptionsKeyValue = channelOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

const calendarTypeOptions = [
  { key: 'bible', display_name: 'Bible' },
  { key: 'gospel', display_name: 'Gospel' },
  { key: 'livestream', display_name: 'Livestream' },
  { key: 'motivation', display_name: 'Motivation' },
  { key: 'movie', display_name: 'Movie' },
  { key: 'music', display_name: 'Music' },
  { key: 'podcast', display_name: 'Podcast' },
  { key: 'preaching', display_name: 'Preaching' },
  { key: 'testimony', display_name: 'Testimony' },
  { key: 'tutorial', display_name: 'Tutorial' },
  { key: 'conference', display_name: 'Conference' },
  { key: 'audiobook', display_name: 'Audiobook' }
]

// arr to obj, such as { CN : "China", US : "USA" }
const calendarTypeKeyValue = calendarTypeOptions.reduce((acc, cur) => {
  acc[cur.key] = cur.display_name
  return acc
}, {})

export default {
  name: 'ComplexTable',
  components: { Pagination },
  directives: { waves },
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: 'success',
        draft: 'info',
        deleted: 'danger'
      }
      return statusMap[status]
    },
    typeFilter(type) {
      return calendarTypeKeyValue[type]
    },
    channelFilter(option) {
      return channelOptionsKeyValue[option]
    }
  },
  data() {
    return {
      tableKey: 0,
      list: null,
      total: 0,
      listLoading: true,
      listQuery: {
        page: 1,
        limit: 20,
        importance: undefined,
        title: undefined,
        type: undefined,
        sort: '+ordinal'
      },
      channels: [],
      channelOptions,
      calendarTypeOptions,
      sortOptions: [{ label: 'Ordinal Ascending', key: '+ordinal' }, { label: 'Ordinal Descending', key: '-ordinal' }],
      statusOptions: ['published', 'draft', 'deleted'],
      showReviewer: false,
      temp: {
        ordinal: undefined,
        updated_at: new Date(),
        localizedname: '',
        media_category: ''
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      rules: {
        channel: [{ required: true, message: 'channel is required', trigger: 'blur' }],
        ordinal: [{ required: true, message: 'ordinal is required', trigger: 'blur' }],
        media_category: [{ required: true, message: 'media category is required', trigger: 'blur' }],
        localizedname: [{ required: true, message: 'title is required', trigger: 'blur' }],
        language_id: [{ required: true, message: 'language id is required', trigger: 'blur' }]
      },
      downloadLoading: false
    }
  },
  created() {

    var bibleChannelUuid = ''

     axios.get('http://localhost:4000/api/v1.3/orgs/64c1fa34-ebe9-425b-ae58-4815d933b01c/channels?language-id=en&offset=1&limit=50', 
        // { params: { type: 'all', }, }
        )
        .then((res) => {
          console.log('Success Response', res.data);
          res.data.result.forEach((channel) => {
            this.channels.push({ uuid: channel.uuid, basename: channel.basename, hash_id: channel.hash_id, updated_at: channel.updated_at });
            console.log('this.channels', this.channels);
            console.log('channel.basename', channel.basename);
            if(channel.basename == 'Bible') {
              console.log('found Bible');
              bibleChannelUuid = channel.uuid
              this.getList(bibleChannelUuid)
            }
          });
        })
        .catch((err) => {
          console.log('Error', err);
        });
    console.log('bibleChannelUuid', bibleChannelUuid);
    // this.getList()
  },
  methods: {
    getList(channelUuid) {
      console.log(`getList: ${channelUuid}`);
      if(channelUuid != null) {
        this.listLoading = true
        return axios.get(`http://localhost:4000/api/v1.3/channels/${channelUuid}/playlists?language-id=en&offset=1&limit=1000`)
          .then(response => {
            console.log(response)
            this.listLoading = false
            this.list = response.data.result
            this.total = response.data.total_entries
          })
      }

    },
    handleSetChannel(channelUuid) {
      console.log(`handleSetChannel: ${channelUuid}`);
      // this.$ELEMENT.size = size
      // this.$store.dispatch('app/setSize', size)
      // this.refreshView()
      if(channelUuid != null) {
        this.getList(channelUuid)
        this.$message({
          message: 'Switch Size Success',
          type: 'success'
        })
      }
    },
    handleFilter() {
      this.listQuery.page = 1
      this.getList()
    },
    handleModifyStatus(row, status) {
      this.$message({
        message: '操作Success',
        type: 'success'
      })
      row.status = status
    },
    sortChange(data) {
      const { prop, order } = data
      if (prop === 'ordinal') {
        this.sortByOrdinal(order)
      }
    },
    sortByOrdinal(order) {
      if (order === 'ascending') {
        this.listQuery.sort = '+ordinal'
      } else {
        this.listQuery.sort = '-ordinal'
      }
      this.handleFilter()
    },
    resetTemp() {
      this.temp = {
        ordinal: undefined,
        updated_at: new Date(),
        localizedname: '',
        media_category: ''
      }
    },
    handleCreate() {
      this.resetTemp()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    createData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          this.temp.id = parseInt(Math.random() * 100) + 1024 // mock a id
          this.temp.author = 'vue-element-admin'
          createArticle(this.temp).then(() => {
            this.list.unshift(this.temp)
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Created Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleUpdate(row) {
      this.temp = Object.assign({}, row) // copy obj
      this.temp.updated_at = new Date(this.temp.updated_at)
      this.dialogStatus = 'update'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    updateData() {
      this.$refs['dataForm'].validate((valid) => {
        if (valid) {
          const tempData = Object.assign({}, this.temp)
          tempData.updated_at = +new Date(tempData.updated_at) // change Thu Nov 30 2017 16:41:05 GMT+0800 (CST) to 1512031311464
          updateArticle(tempData).then(() => {
            for (const v of this.list) {
              if (v.id === this.temp.id) {
                const index = this.list.indexOf(v)
                this.list.splice(index, 1, this.temp)
                break
              }
            }
            this.dialogFormVisible = false
            this.$notify({
              title: 'Success',
              message: 'Update Successfully',
              type: 'success',
              duration: 2000
            })
          })
        }
      })
    },
    handleDelete(row) {
      this.$notify({
        title: 'Success',
        message: 'Delete Successfully',
        type: 'success',
        duration: 2000
      })
      const index = this.list.indexOf(row)
      this.list.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    handleDownload() {
      this.downloadLoading = true
      import('@/vendor/Export2Excel').then(excel => {
        const tHeader = ['updated_at', 'localizedname', 'media_category', 'importance', 'status']
        const filterVal = ['updated_at', 'localizedname', 'media_category', 'importance', 'status']
        const data = this.formatJson(filterVal, this.list)
        excel.export_json_to_excel({
          header: tHeader,
          data,
          filename: 'table-list'
        })
        this.downloadLoading = false
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'updated_at') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    },
    getSortClass: function(key) {
      const sort = this.listQuery.sort
      return sort === `+${key}`
        ? 'ascending'
        : sort === `-${key}`
          ? 'descending'
          : ''
    }
  }
}
</script>
