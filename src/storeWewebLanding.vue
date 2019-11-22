<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div class="store_weweb_landing" :style="customStyle">
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl"></wwSectionEditMenu>
        <!-- wwManager:end -->

        <!--TOP WWOBJS-->
        <div class="top-ww-objs">
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.topWwObjs" class="top-ww-obj" @ww-add="add(section.data.topWwObjs, $event)" @ww-remove="remove(section.data.topWwObjs, $event)">
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>
        </div>

        <div class="ww-popup-store">
            <wwObject class="background" :ww-object="section.data.background" ww-category="background" :class="{'section-background': editMode}"></wwObject>

            <div class="store-wrapper">
                <div :class="{'popup-container-loader': isLoading, 'loading': isLoading}">
                    <div class="loader">
                        <div class="spinner">
                            <div class="arc arc-1"></div>
                            <div class="arc arc-2"></div>
                        </div>
                    </div>
                </div>
                <div class="content">
                    <!-- theme container -->
                    <div class="themes-container">
                        <div class="tabs">
                            <div class="tab active" @click="toggleItem()">themes</div>
                        </div>
                        <!-- 
                        <div class="theme-header">
                            <div class="theme-filter-wrapper">
                                <wwManagerSelect class="select select-theme-fix" :options="filterOptions" :value="filterId" @change="setThemeFromId($event)"></wwManagerSelect>
                                <div class="search search-theme-fix">
                                    <wwManagerSearchBarIcon class="search-bar" placeholder="Portfolio, slider, tools, ..." v-model="search"></wwManagerSearchBarIcon>
                                </div>
                            </div>
                        </div>
                        -->

                        <div class="themes ww-scroll-bar">
                            <div class="theme" v-for="theme in filteredThemes" :key="theme.id" v-show="theme.sectionsCount > 0" @click="selectTheme(theme)" :class="{'active': selectedTheme == theme}">
                                <div class="theme-name">
                                    {{ theme.themeName }}
                                    <span class="section-count">
                                        {{ theme.sectionsCount }}
                                        <span v-if="theme.sectionsCount > 1">sections</span>
                                        <span v-else>Section</span>
                                    </span>
                                </div>
                                <div class="theme-preview-wrapper">
                                    <div class="theme-preview" v-for="(preview, index) in theme.themePreviews" :key="index" :style="{'background-image': 'url('+getThemePreviewUrl(preview, 100)+')'}"></div>
                                </div>
                                <div class="theme-description">{{ theme.description }}</div>
                                <div class="themeTags">{{ theme.tags }}</div>
                            </div>
                        </div>
                    </div>
                </div>

                <!-- theme sections -->
                <div class="sections-wrapper">
                    <wwStoreSections :sections="storeSectionTypes" :categories="categories" @selectSection="selectSection"></wwStoreSections>
                </div>
            </div>

            <div class="mobile-store">
                <div class="dropdown-mobile">
                    <div class="select">
                        <select name="menus-select" v-model="selectedTheme" @change="selectTheme(selected)">
                            <option v-for="theme in filteredThemes" :key="theme.id" :value="theme.themeName" v-show="theme.sectionsCount > 0">
                                <div>{{ theme.themeName }}</div>
                            </option>
                        </select>
                        <div class="text" v-if="selectedTheme && selectedTheme.themeName">{{ selectedTheme.themeName }}</div>
                        <i class="wwi wwi-chevron-down"></i>
                    </div>
                </div>

                <div class="mobile-sections-wrapper">
                    <div class="section" v-for="(section, index) in storeSectionTypes" :key="index" @click="selectSection(section)">
                        <div class="section-wrapper" :style="{'background-image': 'url('+ getPreviewUrl(section)+')'}">
                            <div class="section-preview">
                                <div class="preview-name">{{ section.name }}</div>
                                <div class="preview-category">{{ section.categoryName }}</div>
                                <div class="preview-tag-wrapper">
                                    <div class="preview-tag" v-for="(tag, index) in formatTags(section.tags)" :key="index">{{ '#' + tag }}</div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <wwPopupSectionNamePreview v-if="showSectionName" :section="selectedSection" @close="showSectionName = false"></wwPopupSectionNamePreview>

        <!--BOTTOM WWOBJS-->
        <div class="bottom-ww-objs">
            <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="section.data.bottomWwObjs" class="top-ww-obj" @ww-add="add(section.data.bottomWwObjs, $event)" @ww-remove="remove(section.data.bottomWwObjs, $event)">
                <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<script>

import axios from 'axios'
import wwStoreSections from './components/wwStoreSections.vue'
import wwManagerSearchBarIcon from './components/wwManagerSearchBarIcon.vue'
import wwPopupSectionNamePreview from './components/wwPopupSectionNamePreview.vue'
// import wwMobileStore from './components/wwMobileStore.vue'

export default {
    name: "__COMPONENT_NAME__",
    components: {
        wwStoreSections,
        // wwMobileStore,
        wwManagerSearchBarIcon,
        wwPopupSectionNamePreview
    },
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            dbCategories: null,
            filters: [],
            selectedSection: {},
            filterId: null,
            fixedCategories: null,
            storeSectionTypes: null,
            selectedTheme: undefined,
            loading: false,
            showSectionName: false,
            filterOptions: {
                type: 'text',
                values: [
                    {
                        value: null,
                        default: true,
                        text: {
                            en: 'Filters',
                            fr: 'Filters'
                        }
                    }
                ]
            },
        }
    },
    computed: {
        section() {
            return this.sectionCtrl.get();
        },
        isLoading() {
            return this.loading
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },
        customStyle() {
            return {
                '--activeHeight': `${this.activeHeight + 28}px`
            }
        },

        selected() {
            for (const theme of this.filters) {
                if (this.selectedTheme === theme.themeName)
                    return theme
            }
            return undefined
        },

        categories() {
            return this.fixedCategories
        },
        filteredThemes() {
            if (this.filters && this.filters.length > 0) {
                this.filters.sort(function (a, b) {
                    return b.sectionsCount - a.sectionsCount;
                });
                if (this.filterId)
                    return this.filters.filter(theme => {
                        return theme.themeId == this.filterId
                    })
                else if (this.search) {
                    return this.filters.filter(theme => {
                        if (theme.tags)
                            return theme.tags.toLowerCase().includes(this.search.toLowerCase())
                        else if (theme.description)
                            return theme.description.toLowerCase().includes(this.search.toLowerCase())
                    })

                } else {
                    return this.filters
                }
            }
        },

    },
    async created() {
        try {
            //Initialize section data
            let needUpdate = false
            this.section.data = this.section.data || {};

            if (_.isEmpty(this.section.data.topWwObjs)) {
                this.section.data.topWwObjs = [];
                needUpdate = true;
            }
            if (_.isEmpty(this.section.data.bottomWwObjs)) {
                this.section.data.bottomWwObjs = [];
                needUpdate = true;
            }
            if (!this.section.data.background) {
                this.section.data.background = wwLib.wwObject.getDefault({
                    type: 'ww-color'
                });
                needUpdate = true
            }

            if (needUpdate) {
                this.sectionCtrl.update(this.section);
            }
        } catch (error) {
            console.error(error)
        }
    },
    mounted() {
        this.init()
    },
    beforeDestroy() {
        // window.removeEventListener('scroll', this.onScroll)
    },
    beforeSave() {

    },
    methods: {
        async init() {
            this.loading = true;

            this.dbCategories = await this.getSectionsCategories()

            await this.getAssetsThemes()
            if (this.filteredThemes && this.filteredThemes.length > 0)
                this.selectedTheme = this.filteredThemes[0]

            this.loading = false;
        },
        getText(text) {
            return wwLib.wwLang.getText(text)
        },


        async toggleItem() {
            try {
                this.filterId = null
                this.search = ''
                this.loading = true

                await this.getAssetsThemes()

                this.loading = false
            } catch (error) {
                console.error(error)
            }
        },
        /* get the categories and add an all value to them */
        async getSectionsCategories() {
            try {
                const _url = 'theme/section_categories'

                const _sectionsCategories = await this.apiRequest(_url, 'GET')

                if (_sectionsCategories.data) {
                    this.fixedCategories = JSON.parse(JSON.stringify(_sectionsCategories.data))
                    this.fixedCategories.unshift({ id: 0, name: "all" })

                    this.fixedCategories.sort((a, b) => {
                        if (!a.name) return '1'
                        if (!b.name) return '-1'
                        return a.name.localeCompare(b.name)
                    })
                }
                else
                    throw new Error("NO_CATEGORIES_FOUND")
                return _sectionsCategories.data
            } catch (error) {
                console.error(error)
            }
        },

        /* - get own assets themes and select the first theme - */
        async getAssetsThemes() {
            try {
                this.filters = await this.getMyAssetsThemes()

                if (this.filteredThemes && this.filteredThemes.length > 0) {

                    await this.selectTheme(this.filteredThemes[0])
                    await this.formatThemeForFilter(this.filters)
                } else {
                    console.error("no theme");
                }
            } catch (error) {
                console.error(error)
            }
        },
        /* get user themes, added origin to make the distinction between the theme of the design and the user */
        async getMyAssetsThemes() {
            try {
                const _url = 'theme/get_public_store_theme'

                const _usersThemes = await this.apiRequest(_url, 'GET')

                const _themes = []

                for (const theme of _usersThemes.data) {
                    _themes.push(theme)
                }

                return _themes
            } catch (error) {
                console.error(error)
            }
        },

        async getPublicStoreSectionTypes(themeId) {
            try {
                const _url = `theme/${themeId}/get_public_store_sections`

                const _requestUrl = this.getApiUrl(_url)
                const response = await this.apiRequest(_url, 'GET')
                return response.data
            } catch (error) {
                console.error(error)
            }

        },


        /* - format themes filter option - */
        async formatThemeForFilter(themes) {
            try {
                this.filterOptions.values = [
                    {
                        value: null,
                        default: true,
                        text: {
                            en: 'Filters',
                            fr: 'Filters'
                        }
                    }
                ]
                for (const theme of themes) {
                    this.filterOptions.values.push({
                        value: theme.themeId,
                        text: {
                            en: theme.themeName
                        }
                    })
                }
            } catch (error) {
                console.error(error)
            }

        },

        async selectSection(section) {
            this.showSectionName = true
            this.selectedSection = section

        },

        async selectTheme(theme) {
            if (theme)
                try {
                    this.loading = true
                    this.selectedTheme = theme
                    this.storeSectionTypes = await this.getPublicStoreSectionTypes(theme.themeId)
                    this.loading = false
                    this.$forceUpdate();
                } catch (error) {
                    console.error(error)
                }

        },

        getThemePreviewUrl(imageUrl, size) {
            try {
                let url = `https://i.twic.pics/v1/resize=${size}/`
                if (!imageUrl) {
                    url += wwLib.wwApiRequests._getCdnUrl() + 'public/images/no_preview.jpg';
                } else {
                    if (imageUrl.includes("http"))
                        url += imageUrl
                    else
                        url += wwLib.wwApiRequests._getCdnUrl() + 'developers/' + imageUrl
                }
                return url;
            } catch (error) {
                console.error(error)
            }
        },

        getPreviewUrl(section) {
            try {
                let url = 'https://i.twic.pics/v1/resize=800/'
                if (!section.previews.length) {
                    url += wwLib.wwApiRequests._getCdnUrl() + 'public/images/no_preview.jpg';
                } else {
                    if (section.previews[0].includes("http"))
                        url += section.previews[0];
                    else
                        url += wwLib.wwApiRequests._getCdnUrl() + 'developers/' + section.previews[0]
                }
                return url;
            } catch (error) {
                console.error(error)
            }
        },

        formatTags(tags) {
            try {
                let _tagsArray = []
                const tagsArray = []
                if (tags) {
                    _tagsArray = tags.split(',')
                    for (const tag of _tagsArray) {
                        tagsArray.push(tag.trim())
                    }
                }
                return tagsArray
            } catch (error) {
                console.error(error)
            }
        },


        /* Get api url */
        getApiUrl(url) {
            if (wwLib.envMode == 'development') {
                return 'http://localhost:3000/v1/' + url
                // return 'http://weweb-api:3000/v1/' + url
            } else if (wwLib.envMode == 'preprod') {
                return 'https://api.weweb.dev/v1/' + url
            } else if (wwLib.envMode == 'staging') {
                return 'https://api.weweb.space/v1/' + url
            } else {
                return 'https://api.weweb.app/v1/' + url
            }
        },

        /* - make api request to weweb-api - */
        async apiRequest(url, method) {
            try {
                const _requestUrl = this.getApiUrl(url)
                const result = await axios({
                    method,
                    url: _requestUrl,
                    data: {}
                });
                if (!result) throw new Error("REQUEST_FAILED")
                return result

            } catch (error) {
                console.error(error)
            }

        },

        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },


        /* wwManager:end */
    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->



<style scoped lang="scss">
@import "./assets/manager-css/main";

.store_weweb_landing {
    margin-bottom: 20px;

    .ww-scroll-bar {
        &::-webkit-scrollbar-thumb {
            background-color: #808080;
        }
        &::-webkit-scrollbar-track {
            background-color: #ffffff00;
        }
        &::-webkit-scrollbar {
            width: 5px;
            height: 5px;
            background-color: #ffffff00;
        }
    }
    .popup-container-loader {
        position: absolute;
        overflow: hidden;
        top: 75px;
        left: 0;
        bottom: 0;
        right: 0;
        z-index: 10;
        &.loading {
            background-color: rgba(255, 255, 255, 0.9);
            .loader {
                display: flex !important;
            }
        }

        .loader {
            position: absolute;
            display: flex;
            top: 75px;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(255, 255, 255, 0.5);
            align-items: center;
            justify-content: center;
            z-index: 1;

            .spinner {
                display: inline-block;
                position: relative;
                animation: spinner 1.2s linear infinite;
                width: 80px;
                height: 80px;

                .arc {
                    width: 100%;
                    height: 100%;
                    position: absolute;
                    top: 0;
                    left: 0;
                    display: block;
                    border-radius: 50%;
                    border: 7px solid $ww-blue;
                    animation: arc-colors 6s
                        cubic-bezier(0.05, 0.865, 0.065, 0.99) infinite;

                    &.arc-2 {
                        transform: rotate(180deg);
                        animation: arc-colors-2 6s
                            cubic-bezier(0.05, 0.865, 0.065, 0.99) infinite;
                    }
                }
            }
        }
    }

    @keyframes loading {
        0% {
            transform: translateX(0%);
        }
        100% {
            transform: translateX(100%);
        }
    }

    @keyframes spinner {
        0% {
            transform: rotate(0deg);
        }
        100% {
            transform: rotate(360deg);
        }
    }

    @keyframes arc-colors {
        0% {
            border-color: $ww-blue transparent transparent transparent;
        }
        50% {
            border-color: $ww-orange transparent transparent transparent;
        }
        100% {
            border-color: $ww-blue transparent transparent transparent;
        }
    }

    @keyframes arc-colors-2 {
        0% {
            border-color: $ww-red transparent transparent transparent;
        }
        50% {
            border-color: $ww-green transparent transparent transparent;
        }
        100% {
            border-color: $ww-red transparent transparent transparent;
        }
    }

    .ww-popup-store {
        display: flex;
        flex-direction: column;

        .background {
            position: absolute;
            top: 0;
            left: 0;
            height: 100%;
            width: 100%;
            z-index: -1;
        }

        .section-background {
            pointer-events: all;
            z-index: 0;
        }

        .store-wrapper {
            width: 100%;
            // height: 800px;
            display: none;
            // overflow: hidden;
            padding-top: 20px;
            margin-bottom: 50px;
            pointer-events: all;

            margin-bottom: 50px;

            @media (min-width: 1024px) {
                display: flex;
            }
            .search {
                display: flex;
                justify-content: center;
                .search-bar {
                    width: 100%;
                }
            }
            .content {
                display: flex;
                width: 330px;
                height: 100%;
                float: left;
                position: sticky;
                top: 0;
                padding-top: 50px;

                .themes-container {
                    padding: 0 20px 0 45px;
                    display: flex;
                    flex-direction: column;
                    width: 100%;
                    .tabs {
                        display: flex;
                        justify-content: center;
                        margin-bottom: 20px;
                        .tab {
                            padding: 0px 20px 10px 20px;
                            font-size: 10px;
                            line-height: 13px;
                            text-align: center;
                            text-transform: uppercase;
                            font-family: $ww-font;
                            font-weight: bold;
                            cursor: pointer;
                            color: $ww-grey-light;
                            position: relative;
                            border-bottom: 1px solid $ww-grey-super-light;
                            &::after {
                                content: "";
                                position: absolute;
                                top: 100%;
                                left: 50%;
                                transform: translate(-50%, -50%);
                                height: 10px;
                                width: 10px;
                                background-color: $ww-red-strong;
                                border-radius: 100%;
                                opacity: 0;
                                transition: opacity 0.3s ease;
                            }
                            &:hover {
                                color: $ww-red;
                            }
                            &.active {
                                color: $ww-red;
                                &::after {
                                    opacity: 1;
                                }
                            }
                        }
                        .tab-fix {
                            text-transform: capitalize;
                            font-size: 1rem;
                            line-height: 15px;
                        }
                    }

                    .tabs {
                        padding-right: 20px;
                        .tab {
                            padding: 0px 0px 10px 0px;
                            flex-basis: 50%;
                            font-size: 0.7rem;
                        }
                    }
                    .theme-header {
                        padding-right: 20px;
                        .theme-filter-wrapper {
                            display: flex;
                            align-items: center;
                            flex-direction: row;
                            margin-bottom: 20px;

                            .select-theme-fix {
                                width: 70px;
                                height: 25px;
                                box-sizing: border-box;
                                justify-content: space-evenly;
                                padding: 5px;
                                margin-right: 10px;
                                border: 1px solid $ww-grey-strong;
                                border-radius: 3px;
                                background-color: white;
                            }
                            .search-theme-fix {
                                font-size: 10px;
                                padding: 2px;
                                flex-grow: 1;
                                border-color: $ww-grey-light;
                                ::placeholder {
                                    /* Chrome, Firefox, Opera, Safari 10.1+ */
                                    color: $ww-grey-super-light;
                                    opacity: 1; /* Firefox */
                                }

                                :-ms-input-placeholder {
                                    /* Internet Explorer 10-11 */
                                    color: $ww-grey-super-light;
                                }

                                ::-ms-input-placeholder {
                                    /* Microsoft Edge */
                                    color: $ww-grey-super-light;
                                }
                            }
                        }
                    }

                    .themes {
                        width: 100%;
                        flex-grow: 1;
                        overflow-y: auto;
                        border-right: 1px solid $ww-grey-super-light;
                        padding-right: 20px;
                        .theme {
                            height: 145px;
                            width: 100%;
                            margin-bottom: 20px;
                            padding: 10px 0 0 0;
                            font-family: $ww-font;
                            color: $ww-grey-light;
                            font-size: 1rem;
                            cursor: pointer;
                            border-radius: 3px;
                            transition: all 0.1s ease;
                            text-transform: capitalize;
                            .theme-name {
                                font-weight: bold;
                                color: $ww-grey;
                                margin-bottom: 5px;
                                font-size: 12px;
                                margin-left: 5px;
                                line-height: 15px;
                                text-transform: uppercase;
                                .section-count {
                                    font-size: 0.8rem;
                                    font-weight: normal;
                                    text-transform: lowercase;
                                }
                            }

                            .theme-preview-wrapper {
                                width: 100%;
                                justify-content: space-around;
                                display: inline-flex;
                                flex-direction: row;
                                height: 65px;

                                margin-bottom: 5px;
                                .theme-preview {
                                    width: 110px;
                                    display: flex;
                                    justify-content: center;
                                    align-items: center;
                                    border-radius: 2px;
                                    background-color: white;
                                    box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.07);
                                    background-size: contain;
                                    background-position: center;
                                    background-repeat: no-repeat;
                                    border: 2px solid white;
                                }
                            }
                            .theme-description {
                                color: $ww-grey;
                                font-size: 10px;
                                font-weight: 500;
                                line-height: 13px;
                                margin-bottom: 5px;
                                margin-left: 5px;
                            }
                            .themeTags {
                                color: $ww-grey;
                                margin-left: 5px;
                                font-size: 9px;
                                font-weight: 300;
                                line-height: 11px;
                            }

                            &.active,
                            &:hover {
                                background-color: #eaf2f8;
                            }
                        }
                    }
                }
            }
            .sections-wrapper {
                flex-wrap: wrap;

                width: calc(100% - 330px);
                float: left;
                height: 100%;
            }
        }

        .mobile-store {
            width: 100%;
            min-height: 100px;
            @media (min-width: 1024px) {
                display: none;
            }
            display: flex;
            justify-content: center;
            flex-direction: column;
            pointer-events: all;
            .dropdown-mobile {
                pointer-events: all;
                display: flex;
                justify-content: center;
                width: 100%;
                .select {
                    color: white;
                    select {
                        display: block;
                        width: 100%;
                        border: none;
                        appearance: none;
                        outline: none;
                        // color: white;
                        width: 100%;
                        height: 100%;
                        position: absolute;
                        top: 0;
                        left: 0;
                        background: none;
                        padding: inherit;
                        padding: 2px 50px 2px 25px;
                        cursor: pointer;
                        font-size: 15px;
                        font-weight: bold;
                        line-height: 30px;
                    }
                    i {
                        width: 25px;
                        height: 25px;
                        font-size: 25px;
                    }
                    padding: 2px 20px 2px 25px;
                    position: relative;
                    pointer-events: all;
                    display: -webkit-box;
                    display: -ms-flexbox;
                    display: flex;

                    justify-content: space-between;

                    align-items: center;
                    // color: white;
                    margin: 20px 0;
                    width: 250px;
                    height: 60px;
                    border-radius: 10px;
                    cursor: pointer;
                    background: linear-gradient(
                        90deg,
                        #455fa2 0%,
                        #2d84c1 100%
                    );
                }
            }

            .mobile-sections-wrapper {
                flex-grow: 1;
                overflow-y: auto;
                flex-wrap: wrap;
                font-family: $ww-font;
                display: flex;
                box-sizing: content-box;
                justify-content: space-evenly;
                // justify-content: flex-start;
                // align-content: flex-start;
                .section {
                    flex-basis: 45%;

                    margin-bottom: 25px;
                    cursor: pointer;
                    border-radius: 2px;
                    justify-content: center;
                    align-items: center;

                    .section-wrapper {
                        position: relative;
                        padding-bottom: 56.25%;
                        border-radius: 2px;
                        background-color: white;
                        background-size: contain;
                        background-position: center;
                        background-repeat: no-repeat;
                        border: 2px solid white;
                        width: 100%;
                        height: 100%;
                        box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.15);
                        &:hover {
                            box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.17);
                            .section-preview {
                                visibility: visible;
                                opacity: 1;
                            }
                        }
                    }
                    .section-preview {
                        display: flex;
                        flex-direction: column;
                        position: absolute;
                        min-height: 50px;
                        bottom: 0;
                        left: 0;
                        right: 0;
                        background: linear-gradient(
                            90deg,
                            rgba(29, 151, 127, 0.9) 0%,
                            rgba(44, 195, 183, 0.9) 100%
                        );
                        opacity: 0;
                        padding: 5px 10px 10px 10px;
                        color: white;
                        visibility: hidden;
                        border-radius: 0 0 3px 3px;
                        transition: opacity 0.2s, visibility 0.2s;
                        .preview-name {
                            text-transform: uppercase;
                            font-size: 1.2rem;
                            font-weight: bold;
                            margin-bottom: 10px;
                        }
                        .preview-category {
                            text-transform: uppercase;
                        }
                        .preview-tag-wrapper {
                            display: inline-flex;
                            font-size: 0.7rem;
                            margin-top: 5px;

                            .preview-tag {
                                margin-right: 5px;
                            }
                        }
                    }
                }
            }
        }
    }
}
</style>
<style lang="scss">
@import "./assets/manager-css/main";

.ww-popup-store {
    .content {
        .themes-container {
            .theme-filter-wrapper {
                .select-theme-fix {
                    color: $ww-grey-light;
                    .text {
                        color: $ww-grey-light;
                        font-family: $ww-font;
                        font-size: 0.8rem !important;
                        line-height: 11px;
                        padding: 0px;
                        text-transform: capitalize;
                    }
                    .wwi-chevron-down {
                        font-size: 10px !important;
                    }
                }
            }
        }
    }
}
</style>