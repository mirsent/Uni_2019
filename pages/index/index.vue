<template>
	<view class="content">
		<view class="logo">
			<image src="../../static/logo.jpg" mode="widthFix"></image>
		</view>
		<scroll-view scroll-y :style="{height: scroll_height+'px'}">
			<view class="form">
				<view class="input-group" v-for="(flag, index) in flags" :key="index">
					<i-icon type="flag" color="#ff4c31" size="30" />
					<view class="input-item">
						<input type="text" :value="flag" :data-index="index" @input="flaghange" />
					</view>
					<i-icon type="trash" color="#ccc" size="20" @tap="flagRemove(index)" />
				</view>
				<view class="input-group">
					<i-icon type="brush" color="#ff4c31" size="30" />
					<view class="input-item">
						<input type="text" placeholder="签名" :value="name" @input="nameSign" />
					</view>
					<i-icon type="trash" color="#ccc" size="20" @tap="nameClear" />
				</view>
			</view>
		</scroll-view>
		<canvas class="canvas-element" canvas-id="canvas"></canvas>
		<view class="action-group">
			<view class="flag">
				<input type="text" placeholder="立flag" @input="flagAddChange" />
				<i-icon type="enter" size="30" @tap="flagAdd" />
			</view>
			<view class="share" @tap="save">
				保存分享
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
                scroll_height: '',
				name: '',
				flag_add: '',
				flags: [
					'暴富'
				]
			}
		},
        onReady() {
            uni.getSystemInfo({
            	success: (res) => {
            		let windowHeight = res.windowHeight;
                    
                    let query = uni.createSelectorQuery();
                    query.select(".logo").boundingClientRect();
                    query.select(".action-group").boundingClientRect();
                    query.exec(data => {
                        let top = data[0];
                        let btn = data[1];
                        this.scroll_heightls = windowHeight - top.height - btn.height;
                    });
            	}
            })
        },
		methods: {
			nameSign(e) {
				this.name = e.detail.value;
			},
			nameClear() {
				this.name = '';
			},
			flaghange(e) {
				let index = e.currentTarget.dataset.index;
				let value = e.detail.value;
				this.flags[index] = value;
			},
			flagAddChange(e) {
				this.flag_add = e.detail.value;
			},
			flagAdd() {
				this.flags.splice(this.flags.length, 0, this.flag_add);
			},
			flagRemove(e) {
				this.flags.splice(e, 1);
			},
			save() {
				uni.showLoading({
					title: '生成图片中...',
					mask: false
				});
				let _this = this;
				let height = 600;
				let ctx = uni.createCanvasContext('canvas')
				ctx.setFillStyle('white')
				ctx.fillRect(0, 0, 375, height)

				let imgWidth = 275,
					imgHeight = 860 / 1050 * imgWidth,
					imgX = (375 - imgWidth) / 2;

				ctx.drawImage('../../static/logo.jpg', imgX, 10, imgWidth, imgHeight)

				let fontX = 50,
					fontY = imgHeight + 50,
					lineHeight = 35;

				ctx.setFillStyle('#000')
				ctx.setFontSize(18)

				ctx.setStrokeStyle('#ff4c2e')
				ctx.setLineWidth(2)
				ctx.setLineDash([5, 5], 5);

				let flags = this.flags;
				for (var i = 0; i < flags.length; i++) {
					ctx.fillText((i - 0 + 1) + '. ' + flags[i], fontX, fontY + i * lineHeight)

					ctx.beginPath();
					ctx.moveTo(fontX + 15, fontY + i * lineHeight + 8);
					ctx.lineTo(320, fontY + i * lineHeight + 8);
					ctx.stroke();
				}

				let footerHeight = 100;

				// 虚线
				ctx.setStrokeStyle('#ccc')
				ctx.setLineWidth(1)
				ctx.setLineDash([2, 6], 5);
				ctx.beginPath();
				ctx.moveTo(fontX, height - footerHeight);
				ctx.lineTo(375-fontX, height - footerHeight);
				ctx.stroke();

				let name = this.name ? this.name : 'M';
				ctx.setFontSize(28)
				ctx.fillText(name, fontX, height - footerHeight + 50)

				let date = new Date();
				let sign1 = '/';
				let sign2 = ':';
				let year = date.getFullYear()
				let month = date.getMonth() + 1;
				let day = date.getDate();
				let hour = date.getHours();
				let minutes = date.getMinutes();
				if (month >= 1 && month <= 9) {
					month = "0" + month;
				}
				if (day >= 0 && day <= 9) {
					day = "0" + day;
				}
				if (hour >= 0 && hour <= 9) {
					hour = "0" + hour;
				}
				if (minutes >= 0 && minutes <= 9) {
					minutes = "0" + minutes;
				}
				let currentdate = year + sign1 + month + sign1 + day + " " + hour + sign2 + minutes

				ctx.setFontSize(14)
				ctx.fillText(currentdate, fontX, height - footerHeight + 80)

				let codeWidth = 60,
					codeHeight = 60;

				ctx.drawImage('../../static/code.jpg', 375 - codeWidth - 55, height - footerHeight + 10, codeWidth, codeHeight)

				ctx.setFontSize(12)
				ctx.fillText('扫码立 Flag', 375 - codeWidth - 55, height - footerHeight + 25 + codeHeight)

				ctx.draw(true, () => {
					uni.canvasToTempFilePath({
						canvasId: 'canvas',
						fileType: 'jpg',
						success: function(res) {
							console.log(res.tempFilePath)
							uni.hideLoading()

							uni.saveImageToPhotosAlbum({
								filePath: res.tempFilePath,
								quality: 1,
								success: function() {
									console.log('save success');
									uni.showToast({
										title: '保存成功',
										mask: false,
										duration: 1500
									});
								}
							});
						},
						fail: function(res) {
							console.log(res);
						}
					})
				})
			}
		},
        onShareAppMessage() {
        	return {
        		title: '属于你的2019',
                imageUrl: '../../static/logo.jpg',
        		path: '/pages/index/index'
        	}
        }
	}
</script>

<style>
	.canvas-element {
		position: fixed;
		right: 999px;
		width: 375px;
		height: 600px;
		margin-bottom: 100px;
	}

	.logo {
		text-align: center;
		padding: 15px 0;
	}

	.logo image {
		width: 550upx;
	}

	.form {
		padding: 0 15px;
	}

	.input-group {
		display: flex;
		align-items: center;
		margin-bottom: 12px;
	}

	.input-group i-icon:first-child {
		margin-right: 5px;
	}

	.input-group i-icon:last-child {
		margin-left: 10px;
	}

	.input-item {
		flex: 1;
		font-size: 36upx;
		color: #8f8f94;
		border-bottom: 2px dashed #ff4c31;
	}

	.action-group {
		position: fixed;
		bottom: 0;
		left: 0;
		right: 0;
	}

	.flag {
		margin: 20px 40px;
		display: flex;
		align-items: center;
		border: 1px solid #000;
	}

	.flag i-icon {
		padding: 5px;
	}

	.flag input {
		flex: 1;
		padding: 5px 10px;
	}

	.share {
		color: #fff;
		line-height: 45px;
		text-align: center;
		background-color: #ff4c2e;
	}
</style>
