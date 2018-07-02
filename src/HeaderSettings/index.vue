<template>
  <div class="btn-group" name="HeaderSettings">

    <el-dropdown trigger="click">

      <span class="el-dropdown-link">
        <i class="fa" :class="[usingBak && 'text-info', processingCls || 'fa-cog']"></i>
      </span>

      <el-dropdown-menu slot="dropdown">

          <div>
            <column-group v-for="(columns, groupName) in colGroups"
              ref="colGroups" :key="groupName"
              :group-name="groupName" :columns="columns">
            </column-group>
          </div>

          <div class="pull-right">
            <el-dropdown size="small" split-button type="default" @click="apply()">
              {{ $i18nForDatatable('Apply') }}
              <el-dropdown-menu slot="dropdown">
                <template v-if="supportBackup">
                  <li @click="apply(true)">
                    &nbsp;&nbsp;
                    <a href="#" @click.prevent>
                      <i class="far fa-save"></i>&nbsp;
                      {{ $i18nForDatatable('Apply and backup settings to local') }}
                    </a>
                    &nbsp;&nbsp;
                  </li>
                  <li v-if="usingBak" @click="rmBackup()">
                    &nbsp;&nbsp;
                    <a href="#" @click.prevent>
                      <i class="far fa-trash-alt text-danger"></i>&nbsp;
                      {{ $i18nForDatatable('Clear local settings backup and restore') }}
                    </a>
                    &nbsp;&nbsp;
                  </li>
                </template>
              </el-dropdown-menu>
            </el-dropdown>

            &nbsp;&nbsp;

          </div>

      </el-dropdown-menu>
    </el-dropdown>
    &nbsp;&nbsp;
    
  </div>
</template>
<script>
import ColumnGroup from './ColumnGroup.vue'
import groupBy from 'lodash/groupBy'
import keyGen from '../_utils/keyGen'
import replaceWith from '../_utils/replaceWith'
import { parseStr, stringify, saveToLS, rmFromLS, getFromLS } from '../_utils/localstorage'

export default {
  name: 'HeaderSettings',
  components: { ColumnGroup },
  props: {
    columns: { type: Array, required: true },
    supportBackup: { type: Boolean, required: true }
  },
  data () {
    const origSettings = stringify(this.columns)
    return {
      origSettings,
      usingBak: false, // is using backup
      processingCls: '',
      storageKey: this.supportBackup && keyGen(origSettings)
    }
  },
  created () {
    if (!this.supportBackup) return

    const backup = getFromLS(this.storageKey)
    if (!backup) return // no backup found

    replaceWith(this.columns, backup)
    this.usingBak = true
  },
  mounted () {
    // control dropdown manually (refers to http://jsfiddle.net/rj3k550m/3)
    /*
    const $el = $(this.$el)
    $(this.$refs.dropdownBtn).on('click', this.toggle)
    $(document).on('click', e => {
      $(e.target).closest($el).length || $el.removeClass('open')
    })
    */
  },
  computed: {
    colGroups () {
      return groupBy(
        this.columns.filter(col => col.label || col.title),
        'group'
      )
    },
    drpMenuStyle () {
      const w = Object.keys(this.colGroups).length * 150
      return {
        padding: '10px 10px 0',
        width: `${w + 25}px`,
        left: `-${w - 25}px`
      }
    }
  },
  methods: {
    apply (alsoBackup) {
      //this.toggle()
      this.$refs.colGroups.forEach(colGroup => { colGroup.apply() })
      alsoBackup && this.$nextTick(this.backup)
    },
    backup () {
      saveToLS(this.storageKey, this.columns)
      this.showProcessing()
      this.usingBak = true
    },
    rmBackup () {
      rmFromLS(this.storageKey)
      this.showProcessing()
      this.usingBak = false
      
      replaceWith(this.columns, parseStr(this.origSettings)) // restore
    },
    toggle () {
      $(this.$el).toggleClass('open')
    },
    showProcessing () {
      ['fa-spinner fa-pulse', 'fa-check', ''].forEach((cls, idx) => {
        setTimeout(() => {
          this.processingCls = cls
        }, idx * 1000)
      })
    }
  }
}
</script>
<style>
.-col-group-container {
  border-bottom: 1px solid #ddd;
}
</style>