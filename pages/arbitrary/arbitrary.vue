<template>
	<view class="container">
		<image class="bg" src="@/static/trigo.jpg"></image>
		<view class="review">
			<!-- <view class="triangle-container"> -->
			<image class="triangle" mode="aspectFit" src="@/static/1.gif"></image>
			<!-- </view> -->
			<view class="ipt-view">
				<view class="label">A</view>
				<input class="ipt" v-model="info.A" type="digit">
				<view class="util">deg</view>
			</view>
			<view class="ipt-view">
				<view class="label">B</view>
				<input class="ipt" v-model="info.B" type="digit">
				<view class="util">deg</view>
			</view>
			<view class="ipt-view">
				<view class="label">C</view>
				<input class="ipt" v-model="info.B" type="digit">
				<view class="util">deg</view>
			</view>
			<view class="ipt-view">
				<view class="label">a</view>
				<input class="ipt" v-model="info.a" type="digit">
			</view>
			<view class="ipt-view">
				<view class="label">b</view>
				<input class="ipt" v-model="info.b" type="digit">
			</view>
			<view class="ipt-view">
				<view class="label">c</view>
				<input class="ipt" v-model="info.c" type="digit">
			</view>
			<view class="btn">
				<button class="handel" type="primary" @click="CalculateIt">计算</button>
				<button class="handel" type="primary" @click="StartUp">清空</button>
			</view>
		</view>
	</view>
</template>

<script setup>
	import { onShareAppMessage, onShareTimeline } from '@dcloudio/uni-app'; 
	import {
		ref
	} from 'vue';
	const info = ref({
		A: '',
		B: '',
		C: '',
		a: '',
		b: '',
		c: '',
	})
	let StandardVal, NoOfSF, InputIs, OutputIs, ErrorIs, CountEnts, TypeIs, NumUnits = 10,
		ValW, ValX, ValY, ValZ, AngleIs, SmallCase, MedCase, BigCase, Ambig, Blank, R2deg = 180 / Math.PI,
		D2rad = Math.PI / 180;
	let InValArray = new Array;
	let OutArray = new Array;
	let OutValArray = new Array;
	let OrigsArray = new Array;
	let ValuesIn  = null;
	let SplitString = null;
	let TestForSize  = null;
	let NextDigit = null;
	let SignIs = null;
	let LenLeft = null;
	let LenRight = null;
	// 验证输入合法性
	function GetInputs() {
		let OrigIs;
		CountEnts = 0;
		TypeIs = 0;
		InputIs = info.value.a;// 斜边a
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);//去掉空格和首0
		// 判断输入是不是数字
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs > 0) {
				InValArray[1] = InputIs;
				OrigsArray[1] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 1
			}
		}
		InputIs = info.value.b;
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs > 0) {
				InValArray[2] = InputIs;
				OrigsArray[2] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 2
			}
		}
		InputIs = info.value.c;
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs > 0) {
				InValArray[3] = InputIs;
				OrigsArray[3] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 4
			}
		}
		InputIs = info.value.A;
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs >= 180) {
				ErrorIs = " 任何角度必须小于180度. ";
				return "N"
			}
			if (InputIs > 0) {
				InValArray[4] = InputIs;
				OrigsArray[4] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 8
			}
		}
		InputIs = info.value.B;
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs >= 180) {
				ErrorIs = " 任何角度必须小于180度. ";
				return "N"
			}
			if (InputIs > 0) {
				InValArray[5] = InputIs;
				OrigsArray[5] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 16
			}
		}
		InputIs = info.value.C;
		OrigIs = InputIs;
		InputIs = CutSpaces(InputIs);
		if (TestInput(InputIs) == "N") {
			return "N"
		} else {
			if (InputIs >= 180) {
				ErrorIs = " 任何角度必须小于180度. ";
				return "N"
			}
			if (InputIs > 0) {
				InValArray[6] = InputIs;
				OrigsArray[6] = OrigIs;
				CountEnts++;
				TypeIs = TypeIs + 32
			}
		}
		if (CountEnts > 3) {
			ErrorIs = " 只能输入三个条件. ";
			return "N"
		}
		if (CountEnts < 3) {
			ErrorIs = " 请输入三个条件. ";
			return "N"
		}
		if (TypeIs == 56) {
			ErrorIs = " Data is NOT independent. See Notes. ";
			return "N"
		}
		return "Y"
	}
	function OrderEdges() {
		if (InValArray[1] == InValArray[2] && (InValArray[2] == InValArray[3])) {
			SmallCase = 99;
			MedCase = 99;
			return
		}
		let Small = InValArray[1];
		SmallCase = 1;
		if (Small > InValArray[2]) {
			Small = InValArray[2];
			SmallCase = 2
		}
		if (Small > InValArray[3]) {
			Small = InValArray[3];
			SmallCase = 3
		}
		let Big = InValArray[1];
		BigCase = 0;
		if (Big < InValArray[2]) {
			Big = InValArray[2];
			BigCase = 3
		}
		if (Big < InValArray[3]) {
			Big = InValArray[3];
			BigCase = 6
		}
		MedCase = SmallCase + BigCase;
		switch (MedCase) {
			case 2:
				;
				if (InValArray[2] + InValArray[3] <= InValArray[1]) {
					return "N"
				}
			case 3:
				;
				if (InValArray[2] + InValArray[3] <= InValArray[1]) {
					return "N"
				}
			case 4:
				;
				if (InValArray[1] + InValArray[3] <= InValArray[2]) {
					return "N"
				}
			case 6:
				;
				if (InValArray[1] + InValArray[3] <= InValArray[2]) {
					return "N"
				}
			case 7:
				;
				if (InValArray[1] + InValArray[2] <= InValArray[3]) {
					return "N"
				}
			case 8:
				;
				if (InValArray[1] + InValArray[2] <= InValArray[3]) {
					return "N"
				}
		}
	}
	// 验证两个角相加不能大于等于180度
	function CheckAnomalies() {
		let Msg1 = "两个角度相加不能超过180度. ";
		switch (TypeIs) {
			case 25:
				;
				if ((InValArray[4]) + (InValArray[5]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 26:
				;
				if ((InValArray[4]) + (InValArray[5]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 28:
				;
				if ((InValArray[4]) + (InValArray[5]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 41:
				;
				if ((InValArray[4]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 42:
				;
				if ((InValArray[4]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 44:
				;
				if ((InValArray[4]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 49:
				;
				if ((InValArray[5]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 50:
				;
				if ((InValArray[5]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
			case 52:
				;
				if ((InValArray[5]) + (InValArray[6]) >= 180) {
					ErrorIs = Msg1;
					return "N"
				}
				break;
		}
		return "Y"
	}

	function MakeStandardValues() {
		let Const;
		switch (TypeIs) {
			case 7:
				;
				break;
			case 35:
				;
				InValArray[3] = Math.sqrt((Math.pow(InValArray[1], 2)) + (Math.pow(InValArray[2], 2)) - (2 *
					InValArray[1] * InValArray[2] * Math.cos(InValArray[6] * D2rad)));
				break;
			case 21:
				;
				InValArray[2] = Math.sqrt((Math.pow(InValArray[1], 2)) + (Math.pow(InValArray[3], 2)) - (2 *
					InValArray[1] * InValArray[3] * Math.cos(InValArray[5] * D2rad)));
				break;
			case 14:
				;
				InValArray[1] = Math.sqrt((Math.pow(InValArray[2], 2)) + (Math.pow(InValArray[3], 2)) - (2 *
					InValArray[2] * InValArray[3] * Math.cos(InValArray[4] * D2rad)));
				break;
			case 25:
				;
				InValArray[6] = 180 - InValArray[4] - InValArray[5];
				Const = InValArray[1] / Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 41:
				;
				InValArray[5] = 180 - InValArray[4] - InValArray[6];
				Const = InValArray[1] / Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 49:
				;
				InValArray[4] = 180 - InValArray[5] - InValArray[6];
				Const = InValArray[1] / Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 26:
				;
				InValArray[6] = 180 - InValArray[4] - InValArray[5];
				Const = InValArray[2] / Math.sin(InValArray[5] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 42:
				;
				InValArray[5] = 180 - InValArray[4] - InValArray[6];
				Const = InValArray[2] / Math.sin(InValArray[5] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 4:
				;
				InValArray[4] = 180 - InValArray[5] - InValArray[6];
				Const = InValArray[2] / Math.sin(InValArray[5] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[3] = Const * Math.sin(InValArray[6] * D2rad);
				break;
			case 28:
				;
				InValArray[6] = 180 - InValArray[4] - InValArray[5];
				Const = InValArray[3] / Math.sin(InValArray[6] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				break;
			case 44:
				;
				InValArray[5] = 180 - InValArray[4] - InValArray[6];
				Const = InValArray[3] / Math.sin(InValArray[6] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				break;
			case 52:
				;
				InValArray[4] = 180 - InValArray[5] - InValArray[6];
				Const = InValArray[3] / Math.sin(InValArray[6] * D2rad);
				InValArray[1] = Const * Math.sin(InValArray[4] * D2rad);
				InValArray[2] = Const * Math.sin(InValArray[5] * D2rad);
				break;
			case 11:
				;
				ValX = InValArray[4];
				ValY = InValArray[2];
				ValZ = InValArray[1];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[3] = ValW;
					if (Ambig == "Y") {
						AngleIs = "B"
					}
				}
				break;
			case 19:
				;
				ValX = InValArray[5];
				ValY = InValArray[1];
				ValZ = InValArray[2];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[3] = ValW;
					if (Ambig == "Y") {
						AngleIs = "A"
					}
				}
				break;
			case 13:
				;
				ValX = InValArray[4];
				ValY = InValArray[3];
				ValZ = InValArray[1];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[2] = ValW;
					if (Ambig == "Y") {
						AngleIs = "C"
					}
				}
				break;
			case 37:
				;
				ValX = InValArray[6];
				ValY = InValArray[1];
				ValZ = InValArray[3];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[2] = ValW;
					if (Ambig == "Y") {
						AngleIs = "A"
					}
				}
				break;
			case 22:
				;
				ValX = InValArray[5];
				ValY = InValArray[3];
				ValZ = InValArray[2];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[1] = ValW;
					if (Ambig == "Y") {
						AngleIs = "C"
					}
				}
				break;
			case 38:
				;
				ValX = InValArray[6];
				ValY = InValArray[2];
				ValZ = InValArray[3];
				if (Gcase4() == "N") {
					return "N"
				} else {
					InValArray[1] = ValW;
					if (Ambig == "Y") {
						AngleIs = "B"
					}
				}
				break;
		}
		if (OrderEdges() == "N") {
			ErrorIs = " 输入数据不正确. ";
			return "N"
		}
		return "Y"
	}

	function Gcase4() {
		Ambig = "N";
		AngleIs = " ";
		let ValO;
		let Adiff = ValX - 90;
		if (Math.abs(Adiff) < 1e-10) {
			Adiff = 0
		}
		let ValP = ValY * Math.sin(ValX * D2rad);
		let Pcase = 0;
		if ((Adiff < 0) && ((ValZ) < (ValY * Math.sin(ValX * D2rad)))) {
			Pcase = 1
		}
		if ((Adiff < 0) && (Math.abs(ValZ - ValY * Math.sin(ValX * D2rad)) < 1e-10) && (Pcase == 0)) {
			Pcase = 2
		}
		if ((Adiff < 0) && ((ValZ) >= (ValY)) && (Pcase == 0)) {
			Pcase = 3
		}
		if ((Adiff < 0) && ((ValZ) < (ValY)) && (Pcase == 0)) {
			Ambig = "Y";
			Pcase = 4
		}
		if ((Adiff == 0) && ((ValZ) <= (ValY)) && (Pcase == 0)) {
			Pcase = 1
		}
		if ((Adiff == 0) && ((ValZ) > (ValY)) && (Pcase == 0)) {
			Pcase = 5
		}
		if ((Adiff > 0) && ((ValZ) <= (ValY)) && (Pcase == 0)) {
			Pcase = 1
		}
		if ((Adiff > 0) && ((ValZ) > (ValY)) && (Pcase == 0)) {
			Pcase = 6
		}
		switch (Pcase) {
			case 1:
				;
				ErrorIs = "条件不符合. ";
				return "N";
			case 2:
				;
				ValW = Math.sqrt(Math.pow(ValY, 2) - Math.pow(ValZ, 2));
				return "Y";
			case 3:
				;
				ValO = Math.sqrt(Math.pow(ValZ, 2) - Math.pow(ValP, 2));
				ValW = ValY * Math.cos(ValX * D2rad) + (ValO);
				return "Y";
			case 4:
				;
				ValO = Math.sqrt(Math.pow(ValZ, 2) - Math.pow(ValP, 2));
				ValW = ValY * Math.cos(ValX * D2rad) + (ValO);
				Ambig = "Y";
				return "Y";
			case 5:
				;
				ValW = Math.sqrt(Math.pow(ValZ, 2) - Math.pow(ValY, 2));
				return "Y";
			case 6:
				;
				ValO = Math.sqrt(Math.pow(ValY, 2) - Math.pow(ValP, 2));
				ValW = Math.sqrt(Math.pow(ValZ, 2) - Math.pow(ValP, 2)) - ValO;
				if (ValW <= 0) {
					ErrorIs = " Given conditions are impossible. ";
					return "N";
				} else {
					return "Y"
				}
		}
	}

	function MakeOutputValues() {
		OutArray[1] = InValArray[1];
		OutArray[2] = InValArray[2];
		OutArray[3] = InValArray[3];
		OutArray[7] = InValArray[1]*1 + InValArray[2]*1 + InValArray[3]*1;
		let semip = OutArray[7] / 2;
		OutArray[8] = Math.sqrt(semip * (semip - InValArray[1]*1) * (semip - InValArray[2]*1) * (semip - InValArray[3]*1));
		OutArray[9] = OutArray[8] / semip;
		OutArray[10] = (InValArray[1] * InValArray[2] * InValArray[3]) / (4 * OutArray[8]);
		switch (SmallCase) {
			case 1:
				;
				OutArray[4] = Math.asin(2 * OutArray[8] / (InValArray[2] * InValArray[3])) * R2deg;
				break;
			case 2:
				;
				OutArray[5] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[3])) * R2deg;
				break;
			case 3:
				;
				OutArray[6] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[2])) * R2deg;
				break;
			case 99:
				;
				OutArray[4] = 60;
				OutArray[5] = 60;
				OutArray[6] = 60;
				break
		}
		switch (MedCase) {
			case 99:
				;
				break;
			case 2:
				;
				OutArray[6] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[2])) * R2deg;
				OutArray[4] = 180 - OutArray[5] - OutArray[6];
				break;
			case 3:
				;
				OutArray[5] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[3])) * R2deg;
				OutArray[4] = 180 - OutArray[5] - OutArray[6];
				break;
			case 4:
				;
				OutArray[6] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[2])) * R2deg;
				OutArray[5] = 180 - OutArray[4] - OutArray[6];
				break;
			case 6:
				;
				OutArray[4] = Math.asin(2 * OutArray[8] / (InValArray[2] * InValArray[3])) * R2deg;
				OutArray[5] = 180 - OutArray[4] - OutArray[6];
				break;
			case 7:
				;
				OutArray[5] = Math.asin(2 * OutArray[8] / (InValArray[1] * InValArray[3])) * R2deg;
				OutArray[6] = 180 - OutArray[4] - OutArray[5];
				break;
			case 8:
				;
				OutArray[4] = Math.asin(2 * OutArray[8] / (InValArray[2] * InValArray[3])) * R2deg;
				OutArray[6] = 180 - OutArray[4] - OutArray[5];
				break;
		}
	}

	function LoadOutputValues() {
		// document.UniForm.Message.value = "计算完成 点击[清除]";
		info.value.a = OutValArray[1];
		info.value.b = OutValArray[2];
		info.value.c = OutValArray[3];
		info.value.A = OutValArray[4];
		info.value.B = OutValArray[5];
		info.value.C = OutValArray[6];
		// document.UniForm.Unit7.value = OutValArray[7];
		// document.UniForm.Unit8.value = OutValArray[8];
		// document.UniForm.Unit9.value = OutValArray[9];
		// document.UniForm.Unit10.value = OutValArray[10];
		if (Ambig == "Y") {
			// document.UniForm.Message2.value = "模棱两可。在尝试补角 " + AngleIs
		}
		ValuesIn = "Y"
	}

	function RestoreOriginals() {
		switch (TypeIs) {
			case 7:
				;
				info.value.a = OrigsArray[1];
				info.value.b = OrigsArray[2];
				info.value.c = OrigsArray[3];
				return;
			case 11:
				;
				info.value.a = OrigsArray[1];
				info.value.b = OrigsArray[2];
				info.value.A = OrigsArray[4];
				return;
			case 19:
				;
				info.value.a = OrigsArray[1];
				info.value.b = OrigsArray[2];
				info.value.B = OrigsArray[5];
				return;
			case 35:
				;
				info.value.a = OrigsArray[1];
				info.value.b = OrigsArray[2];
				info.value.C = OrigsArray[6];
				return;
			case 13:
				;
				info.value.a = OrigsArray[1];
				info.value.c = OrigsArray[3];
				info.value.A = OrigsArray[4];
				return;
			case 21:
				;
				info.value.a = OrigsArray[1];
				info.value.c = OrigsArray[3];
				info.value.B = OrigsArray[5];
				return;
			case 37:
				;
				info.value.a = OrigsArray[1];
				info.value.c = OrigsArray[3];
				info.value.C = OrigsArray[6];
				return;
			case 25:
				;
				info.value.a = OrigsArray[1];
				info.value.A = OrigsArray[4];
				info.value.B = OrigsArray[5];
				return;
			case 41:
				;
				info.value.a = OrigsArray[1];
				info.value.A = OrigsArray[4];
				info.value.C = OrigsArray[6];
				return;
			case 49:
				;
				info.value.a = OrigsArray[1];
				info.value.B = OrigsArray[5];
				info.value.C = OrigsArray[6];
				return;
			case 14:
				;
				info.value.b = OrigsArray[2];
				info.value.c = OrigsArray[3];
				info.value.A = OrigsArray[4];
				return;
			case 22:
				;
				info.value.b = OrigsArray[2];
				info.value.c = OrigsArray[3];
				info.value.B = OrigsArray[5];
				return;
			case 38:
				;
				info.value.b = OrigsArray[2];
				info.value.c = OrigsArray[3];
				info.value.C = OrigsArray[6];
				return;
			case 26:
				;
				info.value.b = OrigsArray[2];
				info.value.A = OrigsArray[4];
				info.value.B = OrigsArray[5];
				return;
			case 42:
				;
				info.value.b = OrigsArray[2];
				info.value.A = OrigsArray[4];
				info.value.C = OrigsArray[6];
				return;
			case 50:
				;
				info.value.b = OrigsArray[2];
				info.value.B = OrigsArray[5];
				info.value.C = OrigsArray[6];
				return;
			case 28:
				;
				info.value.c = OrigsArray[3];
				info.value.A = OrigsArray[4];
				info.value.B = OrigsArray[5];
				return;
			case 48:
				;
				info.value.c = OrigsArray[3];
				info.value.A = OrigsArray[4];
				info.value.C = OrigsArray[6];
				return;
			case 52:
				;
				info.value.c = OrigsArray[3];
				info.value.B = OrigsArray[5];
				info.value.C = OrigsArray[6];
				return;
			case 56:
				;
				info.value.A = OrigsArray[4];
				info.value.B = OrigsArray[5];
				info.value.C = OrigsArray[6];
				return;
		}
	}
	// 去掉空格和首0
	function CutSpaces(ThisInput) {
		ThisInput = "" + ThisInput;
		let Temp = "";
		SplitString = ThisInput.split(" ");
		for (let i = 0; i < SplitString.length; i++) {
			Temp += SplitString[i];
		}
		while (Temp.charAt(0) == "0") {
			Temp = Temp.substring(1)
		}
		return Temp
	}
	// 判断输入是不是数字
	function TestInput(ToTest) {
		let TestThis = "" + ToTest,
			Allow = "1234567890-.",
			IsAt;
		if (TestThis == "N") {
			ErrorIs = " 所有的输入必须是数字 ";
			return "N"
		}
		for (let i = 0; i < TestThis.length; i++) {
			let CharIs = TestThis.charAt(i);
			IsAt = Allow.indexOf(CharIs);
			if (IsAt == -1) {
				ErrorIs = "不是有效数字. ";
				return "N"
			}
		}
		TestThis = Number(ToTest);
		if (isNaN(TestThis)) {
			ErrorIs = " 输入不是有效数字. ";
			return "N"
		}
		if (TestThis > 1e9) {
			ErrorIs = " 输入过大! (> 1 000 000 000) ";
			return "N"
		}
		if (TestThis < 1e-5 && TestThis > 0) {
			ErrorIs = " 输入太小! (< 0.000 01) ";
			return "N"
		}
		if (TestThis < 0) {
			ErrorIs = " 不可以是负数! ";
			return "N"
		}
		return "Y";
	}

	function ControlFormat(NumIs) {
		NumIs = Number(NumIs);
		if (Math.abs(NumIs) < 1e-20) {
			NumIs = 0
		}
		if (NumIs == 0) {
			return "0"
		}
		if (NumIs < 0) {
			NumIs = Math.abs(NumIs);
			SignIs = "-";
		} else {
			SignIs = ""
		}
		if (NumIs > 1e15 || NumIs < 1e-7) {
			let Output1 = MakeEnumber(NumIs);
			return SignIs + Output1;
		} else {
			let Output1 = MakeSF(NumIs);
			let Output2 = Format(Output1);
			return SignIs + Output2
		}
	}

	function MakeEnumber(UseNum) {
		let BaseIs, ExpIs, Shift, SignIs, IsE = "N";
		UseNum = "" + UseNum;
		for (let i = 0; i < UseNum.length; i++) {
			let CharIs = UseNum.charAt(i);
			if (CharIs == "e" || CharIs == "E") {
				IsE = "Y";
				break
			}
		}
		if (IsE == "Y") {
			BreakUp = UseNum.split("e");
			ExpIs = BreakUp[1];
			if ((ExpIs) < 0) {
				SignIs = "-"
			} else {
				SignIs = "+"
			}
			ExpIs = Math.abs(Number(ExpIs));
			BaseIs = MakeSF(BreakUp[0]);
			if ((BaseIs) == 10) {
				BaseIs = "1";
				if (SignIs == "-") {
					ExpIs = ExpIs - 1
				} else {
					ExpIs = ExpIs + 1
				}
			}
			ExpIs = SignIs + ExpIs;
			UseNum = BaseIs + "  " + "e" + ExpIs;
		} else {
			ExpIs = 0;
			while (UseNum > 10) {
				UseNum = UseNum / 10;
				ExpIs++;
				Shift = "e+"
			}
			while (UseNum < 1) {
				UseNum = UseNum * 10;
				ExpIs++;
				Shift = "e-"
			}
			BaseIs = MakeSF(UseNum);
			if ((BaseIs) == 10) {
				BaseIs = "1";
				ExpIs = ExpIs + 1
			}
			UseNum = BaseIs + "  " + Shift + ExpIs
		}
		return UseNum
	}

	function MakeSF(UseNum) {
		let CountE = 0,
			Shift = "";
		UseNum = Number(UseNum);
		let AllowError = 1e-10;
		if (Math.abs(UseNum) < AllowError) {
			return 0
		}
		if (Math.abs(UseNum - 1) < AllowError) {
			return 1
		}
		if (Math.abs(UseNum - 10) < AllowError) {
			return 10
		}
		if (Math.abs(UseNum - 100) < AllowError) {
			return 100
		}
		if (Math.abs(UseNum - 1000) < AllowError) {
			return 1000
		}
		if (UseNum > 1 && UseNum < 10) {
			CountE = 0;
			Shift = "N"
		}
		while (UseNum > 10) {
			UseNum = UseNum / 10;
			CountE++;
			Shift = "L"
		}
		while (UseNum < 1) {
			UseNum = UseNum * 10;
			CountE++;
			Shift = "R"
		}
		for (let i = 0; i < NoOfSF - 1; i++) {
			UseNum = UseNum * 10
		}
		UseNum = Math.round(UseNum);
		UseNum = "" + UseNum;
		let NumLength = UseNum.length;
		if (NumLength > NoOfSF) {
			CountE = CountE + 1;
			if (Shift == "R") {
				Shift = "X"
			}
		}
		let BuildNumber = "",
			NextDigit = "";
		if (Shift == "N") {
			for (let i = 0; i < NumLength; i++) {
				NextDigit = UseNum.charAt(i);
				BuildNumber = BuildNumber + NextDigit;
				if (i == CountE) {
					BuildNumber = BuildNumber + "."
				}
			}
		}
		if (Shift == "L") {
			for (var i = 0; i < NumLength; i++) {
				NextDigit = UseNum.charAt(i);
				BuildNumber = BuildNumber + NextDigit;
				if (i == CountE) {
					BuildNumber = BuildNumber + "."
				}
			}
			while (i < CountE + 1) {
				BuildNumber = BuildNumber + "0";
				i++
			}
		}
		if (Shift == "R" && CountE < 13) {
			BuildNumber = "0.";
			for (let i = 0; i < CountE - 1; i++) {
				BuildNumber = BuildNumber + "0"
			}
			BuildNumber = BuildNumber + UseNum.charAt(0);
			for (let i = 1; i < NumLength; i++) {
				BuildNumber = BuildNumber + UseNum.charAt(i)
			}
		}
		if (Shift == "X" && CountE < 13) {
			if (CountE == 2) {
				BuildNumber = "1"
			} else {
				BuildNumber = "0.";
				for (let i = 3; i < CountE; i++) {
					BuildNumber = BuildNumber + "0"
				}
				BuildNumber = BuildNumber + "1"
			}
		}
		let DPisAt = BuildNumber.indexOf(".");
		if (DPisAt > 0) {
			while (BuildNumber.length > DPisAt) {
				if (BuildNumber.charAt(BuildNumber.length - 1) == "0") {
					BuildNumber = BuildNumber.substring(0, BuildNumber.length - 1)
				} else {
					break
				}
			}
		}
		if (BuildNumber.charAt(BuildNumber.length - 1) == ".") {
			BuildNumber = BuildNumber.substring(0, BuildNumber.length - 1)
		}
		return BuildNumber
	}

	function Format(NumToDo) {
		NumToDo = "" + NumToDo;
		TestForSize = Number(NumToDo);
		if (TestForSize > 1e15 || TestForSize < 1e-10) {
			return NumToDo
		}
		let DPisAt = NumToDo.indexOf(".");
		if (DPisAt > 0) {
			while (NumToDo.length > DPisAt) {
				if (NumToDo.charAt(NumToDo.length - 1) == "0") {
					NumToDo = NumToDo.substring(0, NumToDo.length - 1)
				} else {
					break
				}
			}
		}
		if (NumToDo.charAt(NumToDo.length - 1) == ".") {
			NumToDo = NumToDo.substring(0, NumToDo.length - 1)
		}
		DPisAt = NumToDo.indexOf(".");
		if (DPisAt == -1) {
			DPisAt = 999
		}
		let NumLength = NumToDo.length;
		if (DPisAt < NumLength) {
			LenLeft = DPisAt;
			LenRight = NumLength - DPisAt
		} else {
			LenLeft = NumLength;
			LenRight = 0
		}
		let LeftStr = NumToDo.substring(0, LenLeft);
		let RightStr = NumToDo.substring(LenLeft + 1);
		let Separator = " ";
		let NewLeft = "";
		let Count = 0;
		for (let i = LenLeft - 1; i >= 0; i--) {
			NextDigit = LeftStr.charAt(i);
			NewLeft = "" + NextDigit + NewLeft;
			Count++;
			if (Count > 0 && Count % 3 == 0 && !(Count == LenLeft)) {
				NewLeft = Separator + NewLeft
			}
		}
		let NewRight = "";
		Count = 0;
		for (let i = 0; i < LenRight; i++) {
			NextDigit = RightStr.charAt(i);
			NewRight = NewRight + NextDigit;
			Count++;
			if (Count > 0 && Count % 3 == 0 && !(Count == LenRight - 1)) {
				NewRight = NewRight + Separator
			}
		}
		if (LenRight == 0) {
			return NewLeft
		}
		if (LenLeft == 0) {
			return "0." + NewRight
		}
		return NewLeft + "." + NewRight;
	}

	function StartUp() {
		ClearAll();
		// document.UniForm.Message.value = "请输入三个符合的条件，然后点击[计算]";
		// document.UniForm.Message2.value = Blank + Blank + Blank;
		return
	}

	function ClearAll() {
		for (let i = 0; i < NumUnits + 1; i++) {
			OutValArray[i] = "";
			OrigsArray[i] = "";
			InValArray[i] = ""
		}
		Blank = "* * * * * * * * * * ";
		for (let i = 7; i < NumUnits + 1; i++) {
			OutValArray[i] = Blank
		}
		LoadOutputValues();
		ValW = 0;
		ValX = 0;
		ValY = 0;
		ValZ = 0;
		Ambig = "N";
		AngleIs = "";
		ValuesIn = "N";
		return
	}

	function AdjustOutputValues() {
		NoOfSF = 2;
		NoOfSF = NoOfSF + 3;
		if (NoOfSF < 3) {
			NoOfSF = 3
		}
		if (NoOfSF > 9) {
			NoOfSF = 9
		}
		for (let i = 0; i < NumUnits + 1; i++) {
			let Output = ControlFormat(OutArray[i]);
			OutValArray[i] = Output;
		}
		LoadOutputValues();
		RestoreOriginals();
		return
	}
	// 计算方法
	function CalculateIt() {
		if (ValuesIn == "Y") {
			AdjustOutputValues();
			return
		}
		// 验证输入合法性
		if (GetInputs() == "N") {
			uni.showToast({title: ErrorIs, icon: 'none'});
			return
		}
		// 验证角度正确性
		if (CheckAnomalies() == "N") {
			uni.showToast({title: ErrorIs, icon: 'none'});
			return
		}
		if (MakeStandardValues() == "N") {
			uni.showToast({title: ErrorIs, icon: 'none'});
			return
		}
		MakeOutputValues();
		AdjustOutputValues();
		return
	}
	//分享
onShareAppMessage(() => {})
onShareTimeline(() => {})
</script>

<style scoped lang="scss">
	page {
		background: #438bf9;
	}

	.container {
		width: 100%;
		height: 100vh;
		position: relative;
	}

	.bg {
		width: 100%;
		height: 100vh;
	}

	.review {
		position: absolute;
		top: 0;
		right: 0;
		bottom: 0;
		left: 0;
		padding: 0 32rpx;
	}

	.triangle-container {
		width: 50%;
		margin: 30rpx auto 80rpx;
		padding: 0 50rpx;
		background: #fff;
	}

	.triangle {
		width: 50%;
		display: block;
		margin: 50rpx auto;
	}

	.ipt-view {
		display: flex;
		align-items: center;
		color: #fff;
		font-size: 42rpx;
		margin-bottom: 50rpx;
	}

	.label {
		width: 50rpx;
		margin-right: 10rpx;
	}

	.ipt {
		flex: 1;
		border-bottom: solid 1px #fff;
		font-size: 42rpx;
	}

	.util {
		width: 80rpx;
		margin-left: 10rpx;
	}

	.btn {
		// margin-top: 50rpx;
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.handel {
		width: 200rpx;
		height: 80rpx;
		border-radius: 40rpx;
		margin: 0 20rpx;
		background: #007aff;
		line-height: 80rpx;
	}
</style>