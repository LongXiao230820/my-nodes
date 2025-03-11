## form 表单

```js
<u-form ref="formData" 
:label-width="150" label-align="center"   // lable的宽度和对齐方式
:model="formMerchant"  // 数据绑定的对象
:error-type="errorType"   // 表单校验失败后的呈现样式
>
    <view class="bg-white ml-[18rpx] mr-[18rpx] px-2.5 mb-2.5 rounded-lg overflow-hiddern">
    	<u-form-item label="店铺logo" :border-bottom="false" 
			porp="storeLogo"> // 规则校验对应字段
            <u-upload 
                index="logoUpload" // 在各个回调事件中的最后一个参数返回，用于区别是哪一个组件的事件
                :action="action" // 服务器上传地址
                :header="getheader()"  //上传携带的头信息，对象形式
                width="150rpx"
                height="150rpx"
                :max-size="5 * 1024 * 1024" // 上传文件大小限制
                max-count="1" // 最大上传数量限制
                @on-success="uploadSuccess" // 图片上传成功时触发事件
			>
			</u-upload>
        </u-form-item>

        <--  详情地址 -->
        <u-form-item label="详情地址" props="address" :border-bottom="false">
            <view class="flex-1 flex items-center">
                <u-input v-model="formMerchant.address"  // 表单数据绑定
                    :maxlength="60" class="flex-1" placeholder="请输入详情地址" />
                <view class="flex items-center mr-2" 
                    @tap="onSelectAddress"> // @tab就是vue里面的click事件，可替换
                    <image :src="图片地址" :style="{width:'46rpx',height:'46rpx'}" 
                        mode="scaleToFill"  // mode	图片裁剪、缩放的模式 
                        // scaleToFill	不保持纵横比缩放图片，使图片的宽高完全拉伸至填满 image 元素
                    />
                </view>
            </view>
        </u-form-item>

<-- 表单提交 -->
<view>
    <u-button 
		class="w-full rounded-full ml-[18rpx] mr-[18rpx] px-2.5 overflow-hidden"
        :custom-style= "{ // 考虑微信小程序，推荐用custom-style参数修改样式，驼峰
            backgroundColor: '#FF6101'，
            color: '#fff'
        }"
		@click="submit"
	>提交</u-button>
</view>

        <--底部占位误删 解决iphonex 底部bar-->
        <view :style="{width:'100vw',height: safeHeight + 'px'}"></view>
    </view>
</u-form>
<script>
import { getheader } from 'filePath'
import { formRule } from 'filePath'
// form 元素节点
const formData = ref()
const errorType = ref(['message'])  
// 表单数据
const fromMerchant = reactive({
    logo:'',
    address: '',
    location: {
        type: 'Point',
        coordinates: [0, 0],
    }
})


// 图片上传地址
const action: string = import.meta.env.VITE_BASE_URL +import.meta.env.VITE_UPLOAD_URI
const uploadSuccess = (data:any, index: number, lists: any, name: string) {
	// data为服务器返回的数据，name为通过props传递的index参数
    if (data.code === 200) {
        if (name === 'logoUpload') {
            formMerchant.logo = data.data
        }
    } else {
        console.log('上传失败')
    }
}

const latitude = ref(22.535358)
const longitude = ref(113.92379)

onReady(() => { //校验需要在dom创建之后，所以不可以用onLoad 
    formData.value.setRules(formRule)  // 设置表单验证规则
    if (!$useUserStore.getterToken) {
        let routes = getCurrentPages() as any  // UniApp 提供用于获取当前页面栈的信息
        // 页面栈是一个数组，包含了当前打开的所有页面
        let page = routes[routes.length - 1].$page //获取页面栈中最后一个页面（即当前页面）的实例
        const {fullPath} = page // 当前页面完整路径信息
        // #ifndef MP-WEIXIN 
        uni.navigateTo({ // 携带当前页面路径信息，跳转 /pages/sign/SmsAuth 页面
            url: `/pages/sign/SmsAuth?route=${encodeURIComponent(fullPath)}`
        })
        // #endif
        // #ifdef MP-WEIXIN
        uni.navigateTo({
            url: `/pages/sign/wechatAuth?route=${encodeURIComponent(fullPath)}`,
        })
        // #endif
        $settingStore.UPDATE_REFRESH_STATE() // 更新路由状态
    }
})

// 选择详情地址
const onSelectAddress = () => {
    uni.chooseLocation({  // 内置，获取当前的地理位置、速度
        latitude:latitude.value, // 目标纬度
        longitude: longitude.value,  // 目标经度
        success: (res) => {
            if (!res.longitude) return
            latitude.value = res.latitude
            longitude.value = res.longitude
            formMerchant.location.coordinates = [res.longitude,res.latitude]
            formMerchant.address = res.address
        },
        fail:function（err）{
            console.log(err)
        }
    })
}

// 表单提交
const submit = async () => {
    if (formMerchant.location.coordinates?.[0] === 0) {
        return uni.showToast({title:'请在地图上选择准确位置'， icon: 'none'})
    }
    formData.value.validate(async (valid: boolean) => {
        if (valid) {
            const {code,msg} = await doAddShops(formMerchant)
            if (code !== 200) {
                if (code === 100003) {
                    return uni.showToast({
                        title: '已存在同名店铺名称,请修改后重新提交',
                        icon: 'none',
                        duration: 2000,
                    })
                }
                if (code === 80100) {
                    return uni.showToast({
                        title: '提交申请已经存在',
                        icon: 'none',
                        duration: 2000,
                    })
                }
                return uni.showToast({
                    title: '添加失败'，
                    icon:'none',
                    duration: 2000,
                })
            }
            uni.showToast({
                title:'申请店铺成功'，
                icon:'none',
                duration: 2000,
            })
            uni.navigateBack({  // 关闭当前页面，返回上一页面或多级页面
                delta:1, //返回的页面数，如果 delta 大于现有页面数，则返回到首页
            })
        } else {
            return uni.showToast({
                title: '请把表单填写完整'，
                icon: 'none',
                duration: 2000,
            })
        }
    })
}


// 设置底部安全距离
const safeHeight: Ref<number> = ref(0)
uni.getSystemInfo({
    success: (res) => {
        if (res.safeAreaInsets) {
            safeHeight.value = res.safeAreaInsets.bottom
        }
    }
})

// 获取当前用户地址
onLoad(async () => {
    // 安卓开启精确定位
    // #ifdef APP-PLUS  
    const permissionType = 'android.permission.ACCESS_FINE_LOCATION'
    let authFlag = await authorizeUtils.showAuthTipModal(permissionType)
    if (!authFlag) {
        // 权限开启失败提示用户手动打开权限
        authorizeUtils.showManualAuth(permissionType)
        // return
    }
    // #endif
    
    // 获取地图坐标
    // 获取当前位置 
    uni.getLocation({
        type: 'gcj02', //指定了返回的坐标类型
        isHighAccuracy: true, // 是否开启高精确
        geocode: true,  // 是否需要进行地理编码（将坐标转换为地址信息）
        success: function (res) {
            latitude.value = res.latitude
            longitude.value = res.longitude
            console.log(res, '获取当前地址成功')
        }，
        fail：(err) => {
        	console.log(err, '获取当前地址失败')
    	}
    })
})
</script>

// 请求头配置文件，函数返回header对象
export function getheader() {
    const header = {
        'Device-Id': uni.getSystemInfoSync().deviceId,
        'Client-Type': 'CONSUMER',
        'Shop-Id': 0,
        // #ifdef H5
        Platform: 'H5',
        // #endif
        // #ifdef MP-WEIXIN
        //@ts-ignore     //TypeScript 的特殊注释，用于忽略下一行的类型检查错误
        Platform: 'WECHAT_MINI_APP',
        // #endif
        // #ifdef APP-PLUS
        Platform: 'ANDROID',
        // #endif
        Authorization: 'bearer ' + useUserStore().getterToken, 
    }
    return header
}

// 表单校验规则配置文件
export const formRule = {
    logo: [
        {
            required: true,
            message: '请上传店铺logo',
            trigger: ['change', 'blur']
        }
    ]，
    address: [
        {
            required: true,
            message: '请输入详情地址',
            trigger: ['change', 'blur'],
        },
    ],
}
```

## 条件编译

以 **#ifdef** `if defined 仅在某平台存在`  或 **#ifndef** `if not defined 除了某平台均存在`  加 **%PLATFORM%** `平台名称`  