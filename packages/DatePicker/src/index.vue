<template>
    <div name="fv-DatePicker" :class="['fv-' + $theme + '-DatePicker', { disabled: disabled }]">
        <!-- Outside Box -->
        <div class="fv-DatePicker__input" @click="focus()">
            <button v-if="!hideMonth" class="fv-DatePicker__input-month">{{ months[value.getMonth()] }}</button>
            <button v-if="!hideDay" class="fv-DatePicker__input-day" :style="{ borderColor: innerBorderColor }">
                {{ value.getDate() }}
                <span v-if="showWeek">({{ weeks[value.getDay()] }})</span>
            </button>
            <button v-if="!hideYear" :style="{ borderColor: innerBorderColor }" class="fv-DatePicker__input-year">{{
                    value.getFullYear()
            }}</button>
        </div>
        <!-- Popper Box -->
        <div v-show="popper.show" class="fv-DatePicker__input-options" :style="optionsStyle">
            <div class="fv-DatePicker__input-body">
                <div class="fv-DatePicker__input-center-mask" :style="selectStyle" />
                <!-- Month Column -->
                <div v-if="!hideMonth" class="fv-DatePicker__input-options-col" v-hover="hoverUpAndDown" key="col1">
                    <div class="fv-DatePicker__input-options-col-up" @click="clickItem('month', config.buffer - 1)">
                        <i class="ms-Icon ms-Icon--ChevronUp"></i>
                    </div>
                    <div ref="month" :style="style.monthCol" class="fv-DatePicker__input-options-col-items">
                        <div class="fv-DatePicker__input-options-col-item" :class="[selectItemClass]"
                            v-for="(item, index) in options.month" v-hover="hover" :key="`month${item}${index}`"
                            @click="clickItem('month', index)">
                            {{ months[item] }}
                        </div>
                    </div>
                    <div class="fv-DatePicker__input-options-col-down" @click="clickItem('month', config.buffer + 1)">
                        <i class="ms-Icon ms-Icon--ChevronDown"></i>
                    </div>
                </div>
                <!-- Day Column -->
                <div v-if="!hideDay" class="fv-DatePicker__input-options-col" v-hover="hoverUpAndDown" key="col2">
                    <div class="fv-DatePicker__input-options-col-up" @click="clickItem('day', config.buffer - 1)">
                        <i class="ms-Icon ms-Icon--ChevronUp"></i>
                    </div>
                    <div ref="day" :style="style.dayCol" class="fv-DatePicker__input-options-col-items">
                        <div class="fv-DatePicker__input-options-col-item" v-hover="hover" :class="[selectItemClass]"
                            v-for="(item, index) in options.day" :key="`day${item}${index}`"
                            @click="clickItem('day', index)">
                            {{ item > 0 ? item : '' }}
                            <span v-if="showWeek">({{ weeks[weekIndex(item)] }})</span>
                        </div>
                    </div>
                    <div class="fv-DatePicker__input-options-col-down" @click="clickItem('day', config.buffer + 1)">
                        <i class="ms-Icon ms-Icon--ChevronDown"></i>
                    </div>
                </div>
                <!-- Year Column -->
                <div v-if="!hideYear" class="fv-DatePicker__input-options-col" v-hover="hoverUpAndDown" key="col3">
                    <div class="fv-DatePicker__input-options-col-up" @click="clickItem('year', config.buffer - 1)">
                        <i class="ms-Icon ms-Icon--ChevronUp"></i>
                    </div>
                    <div ref="year" class="fv-DatePicker__input-options-col-items">
                        <div class="fv-DatePicker__input-options-col-item" :class="[selectItemClass]"
                            v-for="(item, index) in options.year" v-hover="hover" :key="`year${item}${index}`"
                            @click="clickItem('year', index)">
                            {{ item > 0 ? item : '' }}
                        </div>
                    </div>
                    <div class="fv-DatePicker__input-options-col-down" @click="clickItem('year', config.buffer + 1)">
                        <i class="ms-Icon ms-Icon--ChevronDown"></i>
                    </div>
                </div>
            </div>
            <div class="fv-DatePicker__input-options-bottom-bar">
                <button class="fv-DatePicker__input-options-accept" v-hover="hover" @click="confirm">
                    <i class="ms-Icon ms-Icon--Accept" />
                </button>
                <button class="fv-DatePicker__input-options-cancel" v-hover="hover" @click="cancel">
                    <i class="ms-Icon ms-Icon--Cancel"></i>
                </button>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'FvDatePicker',
    directives: {
        hover: {
            /**
             * @param {HTMLElement} el
             */
            bind(el, { value }) {
                if (value !== undefined && typeof value == 'function') {
                    el.enterFunction = () => {
                        value(true, el);
                    };
                    el.leaveFunction = () => {
                        value(false, el);
                    };
                    el.addEventListener('mouseover', el.enterFunction);
                    el.addEventListener('mouseleave', el.leaveFunction);
                }
            },
            unbind(el) {
                if (el.enterFunction !== undefined && typeof el.enterFunction == 'function') {
                    el.removeEventListener('mouseover', el.hoverFunction);
                    el.removeEventListener('mouseleave', el.leaveFunction);
                }
            },
        },
    },
    props: {
        theme: {
            type: String,
            default: 'system',
        },
        value: {
            type: Date,
            default: () => new Date(1970, 0, 1),
        },
        disabled: {
            type: Boolean,
            default: false,
        },
        hideMonth: {
            type: Boolean,
            default: false,
        },
        hideDay: {
            type: Boolean,
            default: false,
        },
        hideYear: {
            type: Boolean,
            default: false,
        },
        showWeek: {
            type: Boolean,
            default: false,
        },
        // Custom Month
        months: {
            type: Array,
            default: () => ['January', 'February', 'March', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'],
        },
        // Custom Weeks
        weeks: {
            type: Array,
            default: () => ['Sun.', 'Mon.', 'Tue.', 'Wed.', 'Thu.', 'Fri.', 'Sat.'],
        },
        hoverColor: {
            type: String,
            default: undefined,
        },
        optionsStyle: {},
        selectStyle: {},
        innerBorderColor: {
            type: String,
            default: '#cccccc',
        },
        selectItemClass: {},
    },
    data() {
        return {
            popper: {
                show: false,
                month: true,
            },
            selected: {
                date: new Date(this.value),
            },
            options: {
                month: [],
                day: [],
                year: [],
            },
            optionsConfig: {
                month: {
                    slideLock: false,
                },
                day: {
                    slideLock: false,
                },
                year: {
                    slideLock: false,
                },
            },
            config: {
                minDate: new Date(1970, 0, 1),
                maxDate: new Date(2099, 12, 31),
                buffer: 5,
                clickLock: false,
            },
            style: {
                dayCol: {},
                monthCol: {},
                yearCol: {},
            },
            windowEvent: {
                click: (evt) => {
                    // Bubble if out of bounds
                    // close this popper
                    if (!this.popper.show || this.config.clickLock) return;
                    let dom = evt.target;
                    let inside = false;
                    while (dom) {
                        if (dom == this.$el) {
                            inside = true;
                            break;
                        }
                        dom = dom.parentNode;
                    }
                    if (!inside) {
                        this.popper.show = false;
                    }
                },
            },
        };
    },
    computed: {
        $theme() {
            if (this.theme == 'system') return this.$fvGlobal.state.theme;
            return this.theme;
        },
    },
    mounted() {
        this.init();
        // init global window event
        for (let key in this.windowEvent) {
            window.addEventListener(key, this.windowEvent[key]);
        }
    },
    beforeDestroy() {
        if (this.$refs.day !== undefined) this.$refs.day.removeEventListener('scroll', this.optionsConfig.day.scroll);
        if (this.$refs.month !== undefined) this.$refs.month.removeEventListener('scroll', this.optionsConfig.month.scroll);
        if (this.$refs.year !== undefined) this.$refs.year.removeEventListener('scroll', this.optionsConfig.year.scroll);
        for (let key in this.windowEvent) {
            window.removeEventListener(key, this.windowEvent[key]);
        }
    },
    watch: {
        'popper.show'(val) {
            if (val) {
                this.selected.date = new Date(this.value);
                this.$emit('focus');
                this.init();
            }
        },
    },
    methods: {
        init() {
            if (!this.hideMonth) this.setMonthOptions();
            if (!this.hideDay) this.setDayOptions();
            if (!this.hideYear) this.setYearOptions();
        },
        nPrev(num, size, next = 1, offset = 1) {
            let offsetSize = next * size;
            return ((num + offsetSize - offset - next) % size) + offset;
        },
        nNext(num, size, next = 1, offset = 1) {
            let offsetSize = offset * size;
            return ((num + offsetSize + next - offset) % size) + offset;
        },
        setMonthOptions() {
            let month = this.selected.date.getMonth();
            let end = this.nNext(month, 12, this.config.buffer + 1, 0);
            this.options.month = [];
            for (let i = this.nPrev(month, 12, this.config.buffer, 0); i != end; i = this.nNext(i, 12, 1, 0)) {
                this.options.month.push(i);
            }
            this.$nextTick(() => {
                let origin = (this.$refs.month.scrollTop = (this.config.buffer - 4) * 40);
                if (this.optionsConfig.month.scroll) {
                    this.$refs.month.removeEventListener('scroll', this.optionsConfig.month.scroll);
                }
                this.optionsConfig.month.scroll = () => {
                    if (this.optionsConfig.month.slideLock) return;
                    this.slideCol(
                        origin,
                        'month',
                        () => {
                            this.options.month.shift();
                            let month = this.selected.date.getMonth();
                            this.adjustDay(this.selected.date, new Date(this.selected.date.getFullYear(), this.nNext(month, 12, 1, 0)));
                            this.selected.date.setMonth(this.nNext(month, 12, 1, 0));
                            this.options.month.push(this.nNext(month, 12, this.config.buffer + 1, 0));
                        },
                        () => {
                            this.options.month.pop();
                            let month = this.selected.date.getMonth();
                            this.adjustDay(this.selected.date, new Date(this.selected.date.getFullYear(), this.nPrev(month, 12, 1, 0)));
                            this.selected.date.setMonth(this.nPrev(month, 12, 1, 0));
                            this.options.month.unshift(this.nPrev(month, 12, this.config.buffer + 1, 0));
                        }
                    );
                    this.setDayOptions();
                };
                this.$refs.month.addEventListener('scroll', this.optionsConfig.month.scroll);
            });
        },
        setDayOptions() {
            let dayRange = this.dayRange(this.selected.date);
            let day = this.selected.date.getDate();
            let end = this.nNext(day, dayRange, this.config.buffer + 1);
            this.options.day = [];
            for (let i = this.nPrev(day, dayRange, this.config.buffer); i != end; i = this.nNext(i, dayRange)) {
                this.options.day.push(i);
            }
            this.$nextTick(() => {
                let origin = (this.$refs.day.scrollTop = 40 * (this.config.buffer - 4));
                if (this.optionsConfig.day.scroll) {
                    this.$refs.day.removeEventListener('scroll', this.optionsConfig.day.scroll);
                }
                this.optionsConfig.day.scroll = () => {
                    if (this.optionsConfig.day.slideLock) return;
                    this.slideCol(
                        origin,
                        'day',
                        () => {
                            this.options.day.shift();
                            let day = this.selected.date.getDate();
                            this.selected.date.setDate(this.nNext(day, dayRange, 1));
                            this.options.day.push(this.nNext(day, dayRange, this.config.buffer + 1));
                        },
                        () => {
                            this.options.day.pop();
                            let day = this.selected.date.getDate();
                            this.selected.date.setDate(this.nPrev(day, dayRange, 1));
                            this.options.day.unshift(this.nPrev(day, dayRange, this.config.buffer + 1));
                        }
                    );
                };
                this.$refs.day.addEventListener('scroll', this.optionsConfig.day.scroll);
            });
        },
        setYearOptions() {
            let year = this.selected.date.getFullYear() - this.config.buffer;
            let end = year + 2 * this.config.buffer + 1;
            this.options.year = [];
            for (let i = year; i < end; ++i) {
                this.options.year.push(this.adjustYear(i));
            }
            this.$nextTick(() => {
                let origin = (this.$refs.year.scrollTop = (this.config.buffer - 4) * 40);
                if (this.optionsConfig.year.scroll) {
                    this.$refs.year.removeEventListener('scroll', this.optionsConfig.year.scroll);
                }
                this.optionsConfig.year.scroll = async () => {
                    if (this.optionsConfig.year.slideLock) {
                        return;
                    }
                    this.optionsConfig.year.slideLock = true;
                    if (this.$refs.year.scrollTop - origin > 0) {
                        if (this.adjustYear(this.selected.date.getFullYear() + 1) < 0) {
                            this.$refs.year.scrollTop = origin;
                            this.optionsConfig.year.slideLock = false;
                            return;
                        }
                    } else {
                        if (this.adjustYear(this.selected.date.getFullYear() - 1) < 0) {
                            this.$refs.year.scrollTop = origin;
                            this.optionsConfig.year.slideLock = false;
                            return;
                        }
                    }
                    await this.slideCol(
                        origin,
                        'year',
                        () => {
                            this.options.year.shift();
                            let year = this.selected.date.getFullYear();
                            this.adjustDay(this.selected.date, new Date(year + 1, this.selected.date.getMonth()));
                            this.selected.date.setYear(year + 1);
                            this.options.year.push(this.adjustYear(year + this.config.buffer + 1));
                        },
                        () => {
                            this.options.year.pop();
                            let year = this.selected.date.getFullYear();
                            this.adjustDay(this.selected.date, new Date(year - 1, this.selected.date.getMonth()));
                            this.selected.date.setYear(year - 1);
                            this.options.year.unshift(this.adjustYear(year - this.config.buffer - 1));
                        }
                    );
                    this.setDayOptions();
                    this.optionsConfig.year.slideLock = false;
                };
                this.$refs.year.addEventListener('scroll', this.optionsConfig.year.scroll);
            });
        },
        dayRange(date) {
            date = new Date(date);
            date.setDate(1);
            date.setMonth(date.getMonth() + 1);
            date.setDate(0);
            return date.getDate();
        },
        async slideCol(origin, refName, nxtCallback, preCallback) {
            if (Math.abs(this.$refs[refName].scrollTop - origin) >= 20) {
                if (this.$refs[refName].scrollTop > origin) {
                    nxtCallback()
                } else {
                    preCallback()
                }
                return await new Promise(resolve =>
                    this.$nextTick(() => {
                        this.$refs[refName].scrollTop = origin
                        resolve()
                    })
                )
            }
        },
        adjustDay(from, to) {
            let range = this.dayRange(to);
            if (from.getDate() > range) {
                from.setDate(range);
            }
        },
        adjustYear(year) {
            if (year < this.config.minDate.getFullYear() || year > this.config.maxDate.getFullYear()) {
                return -year;
            }
            return year;
        },
        focus() {
            if (this.disabled) return;
            this.popper.show = true;
        },
        confirm() {
            this.$emit('input', new Date(this.selected.date));
            this.$emit('change', new Date(this.selected.date));
            this.popper.show = false;
        },
        cancel() {
            this.popper.show = false;
        },
        async clickItem(colName, index) {
            if (this.config.clickLock) return;
            this.config.clickLock = true;
            // imitation scroll
            this.$refs[colName].scrollTop = (this.config.buffer - 4) * 40;
            let origin = this.$refs[colName].scrollTop;
            this.$refs[colName].scrollTop += index - 5; // init important
            let count = Math.abs(index - 5);
            if (count > 0) {
                let timeout = setInterval(() => {
                    if (this.$refs[colName].scrollTop == origin) {
                        --count;
                        if (!count) {
                            this.config.clickLock = false;
                            clearInterval(timeout);
                            return;
                        }
                    }
                    this.config.scrollLock = true;
                    this.$refs[colName].scrollTop += (index - 5) * 3;
                }, 20);
            } else {
                this.config.clickLock = false;
            }
        },
        weekIndex(day) {
            let date = new Date(this.selected.date);
            date.setDate(day);
            return date.getDay();
        },
        hover(isHover, element) {
            if (this.hoverColor !== undefined) {
                if (isHover) {
                    if (element.hoverStatus === false || element.hoverStatus === undefined) {
                        // store if inner style background is set
                        if (element.style.backgroundColor) element.backgroundColor = element.style.backgroundColor;
                        element.style.backgroundColor = this.hoverColor;
                    }
                } else {
                    // restore inner style background
                    if (element.backgroundColor !== undefined) element.style.backgroundColor = element.backgroundColor;
                    // restore if inner style is not set, set null to use external css
                    else element.style.backgroundColor = null;
                }
                element.hoverStatus = isHover;
            }
        },
        hoverUpAndDown(isHover, element) {
            // find btn.fv-DatePicker__input-options-col-down or btn.fv-DatePicker__input-options-col-up
            let up = element.querySelector('.fv-DatePicker__input-options-col-down');
            let down = element.querySelector('.fv-DatePicker__input-options-col-up');
            if (isHover) {
                if (up.hoverStatus === false || up.hoverStatus === undefined) {
                    // store if inner style background is set
                    if (up.style.backgroundColor) up.backgroundColor = up.style.backgroundColor;
                    up.style.backgroundColor = this.hoverColor;
                }
                if (down.hoverStatus === false || down.hoverStatus === undefined) {
                    // store if inner style background is set
                    if (down.style.backgroundColor) down.backgroundColor = down.style.backgroundColor;
                    down.style.backgroundColor = this.hoverColor;
                }
            } else {
                // restore inner style background
                if (up.backgroundColor !== undefined) up.style.backgroundColor = up.backgroundColor;
                // restore if inner style is not set, set null to use external css
                else up.style.backgroundColor = null;
                if (down.backgroundColor !== undefined) down.style.backgroundColor = down.backgroundColor;
                // restore if inner style is not set, set null to use external css
                else down.style.backgroundColor = null;
            }
            up.hoverStatus = isHover;
            down.hoverStatus = isHover;
        },
    },
};
</script>
