<template>
	<view class="container">
		<image class="bg" src="@/static/trigo.jpg"></image>
		<view class="review">
			<!-- <view class="triangle-container"> -->
				<image class="triangle" mode="aspectFit" src="@/static/view.jpg"></image>
			<!-- </view> -->
			<view class="ipt-view">
				<view class="label">A</view>
				<input class="ipt" v-model="info.A" type="number">
				<view class="util">deg</view>
			</view>
			<view class="ipt-view">
				<view class="label">B</view>
				<input class="ipt" v-model="info.B" type="number">
				<view class="util">deg</view>
			</view>
			<view class="ipt-view">
				<view class="label">a</view>
				<input class="ipt" v-model="info.a" type="number">
			</view>
			<view class="ipt-view">
				<view class="label">b</view>
				<input class="ipt" v-model="info.b" type="number">
			</view>
			<view class="ipt-view">
				<view class="label">c</view>
				<input class="ipt" v-model="info.c" type="number">
			</view>
			<view class="btn">
				<button class="handel" type="primary" @click="compute">计算</button>
				<button class="handel" type="primary" @click="clear">清空</button>
			</view>
		</view>
	</view>
</template>

<script setup>
import { ref } from 'vue';
import { accAdd, accSub, accMul, accDiv } from '@/util'
const info = ref({
	A: '',
	B: '',
	a: '',
	b: '',
	c: '',
})
// 计算
const compute = () => {
	const { A, B, a, b, c} = info.value;
	let temp1 = (A.length > 0 ? 1 : 0) + (B.length > 0 ? 1 : 0);
	let temp2 = (a.length > 0 ? 1 : 0) + (b.length > 0 ? 1 : 0) + (c.length > 0 ? 1 : 0);

	// 排除错误数据
	if ((a.length > 0 && b.length == 0 && c.length == 0 && (temp1 == 2 || temp1 == 0)) ||
		(a.length == 0 && b.length > 0 && c.length == 0 && (temp1 == 2 || temp1 == 0)) ||
		(a.length == 0 && b.length == 0 && c.length > 0 && (temp1 == 2 || temp1 == 0)) ||
		(a.length > 0 && b.length > 0 && c.length > 0)) {
		console.log("数据有误，提供数据过多或过少，无法计算");
		uni.showToast({title: "数据有误，提供数据过多或过少，无法计算", icon: 'none'});
		return;
	}

	// 浮点数筛查
	if ((BASEisNotFloat(a) && a.length > 0) ||
		(BASEisNotFloat(b) && b.length > 0) ||
		(BASEisNotFloat(c) && c.length > 0)) {
		console.log("数据有误，非浮点数，无法计算");
		uni.showToast({title: "数据有误，非浮点数，无法计算", icon: 'none'});
		return;
	}

	if ((BASEisNotFloat(A) && A.length > 0) ||
		(BASEisNotFloat(B) && B.length > 0)) {
		console.log("数据有误，非浮点数，无法计算");
		uni.showToast({title: "数据有误，非浮点数，无法计算", icon: 'none'});
		return;
	}

	// 角度筛查
	if ((parseFloat(A) >= 90 && A.length > 0) ||
		(parseFloat(A) >= 90 && A.length > 0)) {
		console.log("数据有误，角度超出范围，无法计算");
		uni.showToast({title: "数据有误，角度超出范围，无法计算", icon: 'none'});
		return;
	}

	let a_float = parseFloat(a);
	let b_float = parseFloat(b);
	let c_float = parseFloat(c);

	if (b_float >= c_float || a_float >= c_float) {
		console.log("数据有误，直角边大于等于斜边，无法计算");
		uni.showToast({title: "数据有误，直角边大于等于斜边，无法计算", icon: 'none'});
		return;
	}

	let a2 = Math.pow(a_float, 2);
	let b2 = Math.pow(b_float, 2);
	let c2 = Math.pow(c_float, 2);

	// 有2边，求1边和2角度
	if(temp2 == 2) {
		// 没有数据的即是要求的边
		if (a.length == 0) {
			a_float = Math.sqrt(accSub(c2, b2));
			info.value.a = a_float;
		} else if (b.length == 0) {
			b_float = Math.sqrt(accSub(c2, a2));
			info.value.b = b_float;
		} else if (c.length == 0) {
			c_float = Math.sqrt(accAdd(a2, b2));
			info.value.c = c_float;
		}

		let A_float = accMul(Math.asin(a_float / c_float), 180) / Math.PI;
		let B_float = accMul(Math.asin(b_float / c_float), 180) / Math.PI;

		info.value.A = A_float;
		info.value.B = B_float;
	}
	// 有1边1角，求2边1角
	else if(temp2 == 1 && temp1 == 1) {
		let A_float = parseFloat(A);
		let B_float = parseFloat(B);
		// 根据不同边，选用不同公式
		if(a.length > 0) {
			// 已知a边和a角时
			if(A.length > 0) {
				info.value.B = 90 - A_float;
				c_float = accDiv(a_float, Math.sin(A_float * Math.PI / 180));
				b_float = accDiv(a_float, Math.tan(A_float * Math.PI / 180));
				info.value.c = c_float;
				info.value.b = b_float;
			} else { // 已知a边和b角时
				info.value.A = 90 - B_float;
				c_float = accDiv(a_float, Math.cos(B_float * Math.PI / 180));
				b_float = accMul(a_float, Math.tan(B_float * Math.PI / 180));
				info.value.c = c_float;
				info.value.b = b_float;
			}
		} else if(b.length > 0) {
			// 已知b边和a角时
			if(A.length > 0) {
				info.value.B = 90 - A_float;
				c_float = accDiv(b_float, Math.cos(A_float * Math.PI / 180));
				a_float = accMul(b_float, Math.tan(A_float * Math.PI / 180));
				info.value.c = c_float;
				info.value.a = a_float;
			} else { // 已知b边和b角时
				info.value.A = 90 - B_float;
				c_float = accDiv(b_float, Math.sin(B_float * Math.PI / 180));
				a_float = accDiv(b_float, Math.tan(B_float * Math.PI / 180));
				info.value.c = c_float;
				info.value.a = a_float;
			}
		} else {
			// 已知c边和a角时
			if(A.length > 0) {
				info.value.B = accSub(90, A_float);
				a_float = accMul(c_float, Math.sin(A_float * Math.PI / 180));
				b_float = accMul(c_float, Math.cos(A_float * Math.PI / 180));
				info.value.a = a_float;
				info.value.b = b_float;
			} else { // 已知c边和b角时
				info.value.A = accSub(90 - B_float);
				a_float = accMul(c_float, Math.cos(B_float * Math.PI / 180));
				b_float = accMul(c_float, Math.sin(B_float * Math.PI / 180));
				info.value.a = a_float;
				info.value.b = b_float;
			}
		}
	}
	// 其他情况一律认为不合规
	else {
		uni.showToast({title: "数据有误，不予计算", icon: 'none'});
		console.log("数据有误，不予计算");
	}
	console.log("运行完毕");
}
 // 判断是否为浮点数
 function BASEisNotFloat(theFloat) {
	let len = theFloat.length;
	let dotNum = 0;
	if (len == 0)
		return true;
	for (var i = 0; i < len; i++) {
		let oneNum = theFloat.substring(i, i + 1);
		if (oneNum == ".")
			dotNum++;
		if (((oneNum < "0" || oneNum > "9") && oneNum != ".") || dotNum > 1)
			return true;
	}
	if (len > 1 && theFloat.substring(0, 1) == "0") {
		if (theFloat.substring(1, 2) != ".")
			return true;
	}
	return false;
}
// 清空
const clear = () => {
	info.value = {
		A: '',
		B: '',
		a: '',
		b: '',
		c: '',
	}
}
</script>

<style scoped lang="scss">
page{
	background: #438bf9;
}
.container{
	width: 100%;
	height: 100vh;
	position: relative;
}
.bg{
	width: 100%;
	height: 100vh;
}
.review{
	position: absolute;
	top: 0;
	right: 0;
	bottom: 0;
	left: 0;
	padding: 0 32rpx;
}
.triangle-container{
	width: 50%;
	margin: 30rpx auto 80rpx;
	padding: 0 50rpx;
	background: #fff;
}
.triangle{
	width: 50%;
	display: block;
	margin: 50rpx auto;
}
.ipt-view{
	display: flex;
	align-items: center;
	color: #fff;
	font-size: 42rpx;
	margin-bottom: 50rpx;
}
.label{
	width: 50rpx;
	margin-right: 10rpx;
}
.ipt{
	flex: 1;
	border-bottom: solid 1px #fff;
	font-size: 42rpx;
}
.util{
	width: 80rpx;
	margin-left: 10rpx;
}
.btn{
	// margin-top: 50rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}
.handel{
	width: 200rpx;
	height: 80rpx;
	border-radius: 40rpx;
	margin: 0 20rpx;
	background: #007aff;
	line-height: 80rpx;
}
</style>