<template>
	<view class="u-numberbox">
		<view
			class="u-icon-minus"
			@tap="minus"
			:class="{ 'u-icon-disabled': disabled || inputVal <= min }"
			:style="{
				background: bgColor,
				height: inputHeight + 'rpx',
				color: color
			}"
		>
			<u-icon name="minus" :size="size"></u-icon>
		</view>
		<input
			:disabled="disabled"
			:class="{ 'u-input-disabled': disabled }"
			v-model="inputVal"
			class="u-number-input"
			@blur="onBlur"
			type="number"
			:style="{
				color: color,
				fontSize: size + 'rpx',
				background: bgColor,
				height: inputHeight + 'rpx',
				width: inputWidth + 'rpx'
			}"
		/>
		<view
			class="u-icon-plus"
			@tap="plus"
			:class="{ 'u-icon-disabled': disabled || inputVal >= max }"
			:style="{
				background: bgColor,
				height: inputHeight + 'rpx',
				color: color
			}"
		>
			<u-icon name="plus" :size="size"></u-icon>
		</view>
	</view>
</template>

<script>
export default {
	props: {
		// 预显示的数字
		value: {
			type: Number,
			default: 1
		},
		// 背景颜色
		bgColor: {
			type: String,
			default: '#F2F3F5'
		},
		// 最小值
		min: {
			type: Number,
			default: 0
		},
		// 最大值
		max: {
			type: Number,
			default: 99999
		},
		// 步进值，每次加或减的值
		step: {
			type: Number,
			default: 1
		},
		// 是否禁用加减操作
		disabled: {
			type: Boolean,
			default: false
		},
		// input的字体大小，单位rpx
		size: {
			type: [Number, String],
			default: 26
		},
		// 加减图标的颜色
		color: {
			type: String,
			default: '#323233'
		},
		// input宽度，单位rpx
		inputWidth: {
			type: [Number, String],
			default: 80
		},
		// input高度，单位rpx
		inputHeight: {
			type: [Number, String],
			default: 50
		},
		// index索引，用于列表中使用，让用户知道是哪个numberbox发生了变化，一般使用for循环出来的index值即可
		index: {
			type: [Number, String],
			default: ''
		}
	},
	watch: {
		value(val) {
			this.inputVal = +val;
		},
		inputVal(v1, v2) {
			// 为了让用户能够删除所有输入值，重新输入内容，删除所有值后，内容为空字符串
			if(v1 == '') return ;
			let value = 0;
			// 首先判断是否正整数，并且在min和max之间，如果不是，使用原来值
			let tmp = /(^\d+$)/.test(v1) && value[0] != 0;
			if(tmp && v1 >= this.min && v1 <= this.max) value = v1;
			else value = v2;
			this.handleChange(value, 'change');
			this.$nextTick(() => {
				this.inputVal = value;
			})
		}
	},
	data() {
		return {
			inputVal: 0 // 输入框中的值，不能直接使用props中的value，因为应该改变props的状态
		};
	},
	created() {
		this.inputVal = +this.value;
	},
	methods: {
		minus() {
			this.computeVal('minus');
		},
		plus() {
			this.computeVal('plus');
		},
		// 为了保证小数相加减出现精度溢出的问题
		calcPlus(num1, num2) {
			let baseNum, baseNum1, baseNum2;
			try {
				baseNum1 = num1.toString().split('.')[1].length;
			} catch (e) {
				baseNum1 = 0;
			}
			try {
				baseNum2 = num2.toString().split('.')[1].length;
			} catch (e) {
				baseNum2 = 0;
			}
			baseNum = Math.pow(10, Math.max(baseNum1, baseNum2));
			let precision = baseNum1 >= baseNum2 ? baseNum1 : baseNum2; //精度
			return ((num1 * baseNum + num2 * baseNum) / baseNum).toFixed(precision);
		},
		// 为了保证小数相加减出现精度溢出的问题
		calcMinus(num1, num2) {
			let baseNum, baseNum1, baseNum2;
			try {
				baseNum1 = num1.toString().split('.')[1].length;
			} catch (e) {
				baseNum1 = 0;
			}
			try {
				baseNum2 = num2.toString().split('.')[1].length;
			} catch (e) {
				baseNum2 = 0;
			}
			baseNum = Math.pow(10, Math.max(baseNum1, baseNum2));
			let precision = baseNum1 >= baseNum2 ? baseNum1 : baseNum2;
			return ((num1 * baseNum - num2 * baseNum) / baseNum).toFixed(precision);
		},
		computeVal(type) {
			uni.hideKeyboard();
			if (this.disabled) return;
			let value = 0;
			// 减
			if (type === 'minus') {
				value = this.calcMinus(this.inputVal, this.step);
			} else if (type === 'plus') {
				value = this.calcPlus(this.inputVal, this.step);
			}
			// 判断是否小于最小值和大于最大值
			if (value < this.min || value > this.max) {
				return;
			}
			this.inputVal = value;
			this.handleChange(value, type);
		},
		// 处理用户手动输入的情况
		onBlur(event) {
			let val = 0;
			let value = event.detail.value;
			// 如果为非0-9数字组成，或者其第一位数值为0，直接让其等于min值
			// 这里不直接判断是否正整数，是因为用户传递的props min值可能为0
			if(!/(^\d+$)/.test(value) || value[0] == 0) val = this.min;
			val = +value;
			if (val > this.max) {
				val = this.max;
			} else if (val < this.min) {
				val = this.min;
			}
			this.$nextTick(() => {
				this.inputVal = val;
			})
			this.handleChange(val, "blur");
		},
		handleChange(value, type) {
			if (this.disabled) return;
			this.$emit(type, {
				value: value,
				index: this.index
			})
		}
	}
};
</script>

<style lang="scss" scoped>
.u-numberbox {
	display: inline-flex;
	align-items: center;
}

.u-number-input {
	position: relative;
	text-align: center;
	padding: 0;
	margin: 0 6rpx;
	display: flex;
	align-items: center;
	justify-content: center;
}

.u-icon-plus,
.u-icon-minus {
	width: 60rpx;
	display: flex;
	justify-content: center;
	align-items: center;
}

.u-icon-plus {
	border-radius: 0 8rpx 8rpx 0;
}

.u-icon-minus {
	border-radius: 8rpx 0 0 8rpx;
}

.u-icon-disabled {
	color: #c8c9cc !important;
	background: #f7f8fa !important;
}

.u-input-disabled {
	color: #c8c9cc !important;
	background-color: #f2f3f5 !important;
}
</style>
