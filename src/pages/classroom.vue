<style lang="less">
  @import "./src/less/config.less";
  page {
    font-size: 0.8rem;
    background: @bg-color;
    color: #555;
  }
  .picker {
    padding: 0.5rem;
    display: flex;
    justify-content: space-between;
  }
  .table {
    padding: 0.5rem;
    .row {
      background: #efefef;
      box-shadow: 4rpx 4rpx 8rpx #ddd;
      &:nth-child(1) {
        background: #efefef;
      }
      margin-top: 0.5rem;
      display: flex;
      view {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        margin: 0.2rem;
      }
      .item {
        padding: 0.2rem 0;
        color: #fffafa;
        border-radius: 0.1rem;
        view {
          padding: 0.3rem 0;
        }
      }
      .use {
        box-shadow: 4rpx 4rpx 4rpx #ccc;
        background: red;
      }
      .not-use {
        box-shadow: 4rpx 4rpx 4rpx #ccc;
        background: #008e59;
      }
    }
  }
</style>

<template>
  <view class="classroom">
    <picker mode="multiSelector" range-key="name" @change="changeBuilding" @columnchange="columnchange" value="{{index}}" range="{{buildings}}">
      <view class="picker">
        <view>当前选择：{{building.addr}}-{{building.name}}</view>
        <view>
          点击切换
        </view>
      </view>
    </picker>
    <view class="table">
      <view class="thead row">
        <view>教室</view>
        <view>一大节</view>
        <view>二大节</view>
        <view>三大节</view>
        <view>四大节</view>
        <view>五大节</view>
      </view>
      <block wx:for="{{rooms}}" wx:key="{{index}}">
        <view class="row">
          <view class="room-name">{{item.roomName}}</view>
          <block wx:for="{{item.classUse}}" wx:for-item="val" wx:key="{{index}}">
            <view @tap="show({{item}},{{index}})" class="item">
              <view class="{{val.use == 0 ? 'not-use' : 'use'}}">{{val.kcm ? val.kcm[2] : '空'}}</view>
            </view>
          </block>
        </view>
      </block>
    </view>
  </view>
</template>

<script>
  import wepy from 'wepy'
  import HttpMixin from "../mixins/http";
  import ToastMixin from "../mixins/toast";
  import db from "../util/db"
  export default class Classroom extends wepy.page {
    config = {
      navigationBarTitleText: '我要自习'
    };
    data = {
      JA: [{
          name: '一教A座',
          value: 'yjA'
        },
        {
          name: '一教B座',
          value: 'yjB'
        },
        {
          name: '一教C座',
          value: 'yjC'
        },
        {
          name: '一教D座',
          value: 'yjD'
        },
        {
          name: '综合楼B座',
          value: 'zongB'
        },
        {
          name: '综合楼C座',
          value: 'zongC'
        },
        {
          name: '文科楼一区',
          value: 'wen1'
        },
        {
          name: '文科楼二区',
          value: 'wen2'
        }, {
          name: '文科楼三区',
          value: 'wen3'
        }
      ],
      WJ: [{
          name: '东二教',
          value: 'WJdong2'
        },
        {
          name: '东三教',
          value: 'WJdong3'
        },
        {
          name: '基教楼A座',
          value: 'WJjijiaoA'
        },
        {
          name: '基教楼C座',
          value: 'WJjijiaoC'
        }
      ],
      HX: [{
          name: '九教',
          value: 'HX9'
        },
        {
          name: '十教',
          value: 'HX10'
        }
      ],
      building: {
        addr: '望江',
        name: '东二教',
        value: 'WJdong2'
      },
      buildings: [
        [{
            name: '望江',
            value: 'WJ'
          },
          {
            name: '江安',
            value: 'JA'
          },
          {
            name: '华西',
            value: 'HX'
          }
        ],
        [{
            name: '东二教',
            value: 'WJdong2'
          },
          {
            name: '东三教',
            value: 'WJdong3'
          },
          {
            name: '基教楼A座',
            value: 'WJjijiaoA'
          },
          {
            name: '基教楼C座',
            value: 'WJjijiaoC'
          }
        ]
      ],
      index: [0, 0],
      rooms: []
    }
    mixins = [HttpMixin, ToastMixin]
    components = {}
    methods = {
      changeBuilding(e) {
        this.index = e.detail.value
        this.building = this.buildings[1][this.index[1]]
        this.building.addr = this.buildings[0][this.index[0]].name
        this.get(this.building.value)
      },
      columnchange(e) {
        if (e.detail.column !== 0) {
          return
        }
        let building = this.buildings[0][e.detail.value].value
        this.buildings[1] = this[building]
      },
      show(item, i) {
        wepy.showModal({
          title: this.building.name + item.roomName,
          content: `座位数: ${item.roomZws}, 第${i + 1}大节: ${item.classUse[i].kcm || '空闲'}`,
          showCancel: false
        })
      }
    }
    async get(classroom) {
      const resp = await this.POST('/classroom', {
        classroom: classroom
      })
      this.rooms = JSON.parse(resp.data).roomdata
      this.$apply()
      db.Set('classroom', this.building)
    }
    async onLoad() {
      this.building = db.Get('classroom') || this.building
      // 获取教室
      this.get(this.building.value || 'WJdong2')
    }
    onShareAppMessage(options) {
      return {
        title: '一起来自习'
      }
    }
  }
</script>
