<template>
	<div
		:class="['fv-'+$theme+'-Combobox', {disabled: isDisabled}]"
		:style="{background: background, padding: borderWidth, 'border-radius': `${borderRadius}px`, 'z-index': status ? 3 : '', overflow: 'visible'}"
	>
        <div class="combobox-container" @click="status = !isDisabled ? !status : false" :style="{background: inputBackground, 'border-radius': `${borderRadius}px`}">
            <fv-reveal-container
                :parent="() => $el"
                class="fv-combobox-reveal-container"
                :backgroundColor="backgroundLightColor"
                :borderColor="borderLightColor"
                :backgroundGradientSize="120"
                :borderGradientSize="80"
                :borderWidth="borderWidth"
                :borderRadius="borderRadius"
                :disabled="isDisabled"
            ></fv-reveal-container>
            <input class="input" :placeholder="placeholder" readonly :value="valueTrigger(thisValue.text)" :style="{color: inputForeground, 'border-radius': `${borderRadius}px`}"/>
		    <i class="ms-Icon right-icon" :class="[`ms-Icon--${dropDownIcon}`]" :style="{color: dropDownIconForeground}"></i>
        </div>
		<transition name="fv-combobox">
            <div v-show="status" class="combobox-item-container" :style="{background: background, overflow: 'auto'}" ref="co_items">
                <div
                    v-for="(item, index) in options"
                    class="fv-combobox-item"
                    :class="{hr:valueTrigger(item.type) == 'divider', normal: (valueTrigger(item.type) == 'default' || valueTrigger(item.type) == undefined) && !valueTrigger(item.disabled), disabled: valueTrigger(item.disabled), choose: item === thisValue, title: valueTrigger(item.type) == 'header'}"
                    :style="{background: item === thisValue ? choosenBackground : '', color: valueTrigger(item.type) === 'header' ? titleForeground : ''}"
                    @click="Choose($event, item)"
                    :key="index"
                    :title="valueTrigger(item.text)"
                >
                    <slot :item="item">
                        <i class="before-choosen" :style="{background: choosenSliderBackground}"></i>
                        {{valueTrigger(item.type) !== 'divider' ? valueTrigger(item.text) : ''}}
                    </slot>
                </div>
            </div>
        </transition>
	</div>
</template>

<script>
export default {
	name: "FvCombobox",
	props: {
		value: {
			default: () => {
				return {};
			}
		},
		options: {
			default: () => []
		},
		borderWidth: {
			default: 2
		},
		placeholder: {
			default: "Combobox"
		},
		borderRadius: {
			default: "3"
        },
        background: {
            default: ""
        },
        choosenBackground: {
            default: ""
        },
        choosenSliderBackground: {
            default: ""
        },
		inputForeground: {
			default: ""
        },
        inputBackground: {
            default: ""
        },
        titleForeground: {
            default: ""
        },
		dropDownIcon: {
			default: "ChevronDown"
		},
		dropDownIconForeground: {
			default: ""
        },
        disabled: {
            default: false
        },
		theme: {
			type: String,
			default: "system"
		}
	},
	data() {
		return {
			thisValue: this.value,
			status: false
		};
	},
	watch: {
		value(val) {
			this.thisValue = val;
		},
		thisValue(val) {
			this.$emit("input", val);
		}
	},
	computed: {
        isDisabled () {
            return (
                this.options.length == 0 ||
                this.disabled.toString() == "true" ||
                this.disabled == "disabled" ||
                this.disabled === ""
            );
        },
		borderLightColor () {
            if(this.$theme == 'light') {
                return 'rgba(121, 119, 117, 0.6)';
            }
            if(this.$theme == 'dark' || this.$theme == 'custom') {
                return 'rgba(255, 255, 255, 0.6)';
            }
            return 'rgba(121, 119, 117, 0.6)';
        },
        backgroundLightColor () {
            if(this.$theme == 'light') {
                return 'rgba(121, 119, 117, 0.3)';
            }
            if(this.$theme == 'dark' || this.$theme == 'custom') {
                return 'rgba(255, 255, 255, 0.3)';
            }
            return 'rgba(121, 119, 117, 0.3)';
        },
		$theme() {
			if (this.theme == "system") return this.$fvGlobal.state.theme;
			return this.theme;
		}
	},
	mounted () {
		this.outSideClickInit();
	},
	methods: {
		outSideClickInit() {
			window.addEventListener("click", event => {
				let x = event.target;
				let _self = false;
				while (x && x.tagName && x.tagName.toLowerCase() != "body") {
					if (x == this.$el) {
						_self = true;
						break;
					}
					x = x.parentNode;
				}
				if (!_self) this.status = false;
			});
            window.addEventListener("touchend", event => {
				let x = event.target;
				let _self = false;
				while (x && x.tagName && x.tagName.toLowerCase() != "body") {
					if (x == this.$el) {
						_self = true;
						break;
					}
					x = x.parentNode;
				}
				if (!_self) this.status = false;
			});
		},
        valueTrigger (val) {
            if(typeof(val) === 'function')  return val();
            return val;
        },
		Choose (event, item) {
            if(this.valueTrigger(item.disabled))
                return 0;
            if(this.valueTrigger(item.type) == 'header' || this.valueTrigger(item.type) == 'divider')
                return 0;
			this.thisValue = item;
            let target = event.target;
            while(target.getAttribute("class").indexOf("fv-combobox-item") < 0) {
                target = target.parentNode;
            }
            this.$refs.co_items.scrollTop = target.offsetTop;
            this.status = false;
			this.$emit("choose-item", this.thisValue);
		}
	}
};
</script>
