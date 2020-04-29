<template>
	<view>
		<button open-type="getPhoneNumber" @getphonenumber="onGetPhoneNumber">获取手机号码</button>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				appId: uni.getAccountInfoSync().miniProgram.appId,
				sessionKey: "",
				openId:""
			}
		},
		onLoad() {
			this.login()
		},
		methods: {
			// 登录
			login(e) {
				uni.login({
					success: (res) => {
						console.log("login", res);
						if (res.errMsg == "login:ok") {
							var code = res.code
							var AppSecret = "a3ef17cb837510c85210f40bccc384f3"
							var url = "https://api.weixin.qq.com/sns/jscode2session?appid=" +
								this.appId + "&secret=" + AppSecret + "&js_code=" +
								code + "&grant_type=authorization_code"
							uni.request({
								url:url,
								success:(res)=> {
									console.log(res)
									if(res.errMsg=="request:ok"){
										this.sessionKey = res.data.session_key
										this.openId = res.data.openid
									}
								}
							})
						}
					}
				})
			},
			// 获取手机号
			onGetPhoneNumber(e) {
				if (this.sessionKey == "") {
					uni.showModal({
						title: "获取登录信息失败！"
					})
					return
				}
				if (e.detail.errMsg == "getPhoneNumber:ok") {
					var WXBizDataCrypt = require('../../static/js/WXBizDataCrypt')
					var appId = this.appId
					var sessionKey = this.sessionKey
					var encryptedData = e.detail.encryptedData
					var iv = e.detail.iv
					var pc = new WXBizDataCrypt(appId, sessionKey)
					var data = pc.decryptData(encryptedData, iv)
					console.log('解密后 data: ', data)
					uni.showModal({
						title:"手机号码",
						content:data.phoneNumber
					})
				}
			}
		}
	}
</script>
