<template>
    <div class="ww-popup-store">
        <div class="content">
            <div class="categories-container">
                <div class="section-title">
                    <div>
                        <span>sections</span>
                        <span class="section-count">{{ sectionCount }}</span>
                    </div>

                    <div class="search">
                        <wwManagerSearchBarIcon class="search-bar" placeholder="Portfolio, slider, tools, ..." v-model="search"></wwManagerSearchBarIcon>
                    </div>
                </div>
                <div class="filter-sections">
                    <div class="categories">
                        <div class="category" v-for="(category, index) in categories" :key="index" @click="selectCategory(category)" :class="{'active': selectedCategory == category}">{{ category.name }}</div>
                    </div>
                </div>
            </div>

            <div class="sections ww-scroll-bar">
                <div class="section" v-for="(section, index) in filtredSections" :key="index" @click="selectSection(section)">
                    <div class="section-wrapper" :style="{'background-image': 'url('+getPreviewUrl(section)+')'}">
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
</template>

<script> 

import wwManagerSearchBarIcon from './wwManagerSearchBarIcon.vue'


export default {
    name: "wwStoreSections",
    components: {
        wwManagerSearchBarIcon
    },
    props: {
        sections: Array,
        categories: Array,
    },
    data() {
        return {
            search: '',
            stores: [],

            selectedStore: null,
            selectedCategory: null,

            themeSection: null,
            filterThemes: null
        };
    },
    watch: {
        categories: function (newVal, oldVal) { // watch it
            this.selectCategory(newVal[0])
        },
        sections: function (newVal, oldVal) { // watch it
            this.selectCategory(this.categories[0])
        }
    },
    computed: {
        filtredSections() {
            if (!this.sections) return []
            let sections = []
            if (this.selectedCategory && (this.selectedCategory.name.toLowerCase()) == "all") {
                if (this.search && this.sections) {
                    sections = this.sections.filter(section => {
                        return this.searchFilter(section, this.search)
                    })
                } else
                    sections = this.sections
            }
            else {
                sections = this.sections.filter(section => {
                    if (this.selectedCategory)
                        if (this.search) {
                            return (section.sectionCategoryId == this.selectedCategory.id && this.searchFilter(section, this.search))
                        }
                        else
                            return (section.sectionCategoryId == this.selectedCategory.id)
                    else if (this.search) {
                        return this.searchFilter(section, this.search)
                    }
                    else {
                        return this.sections
                    }
                })
            }
            return sections.sort((s1, s2) => { return s1.name.toLowerCase() > s2.name.toLowerCase() ? 1 : -1; })
        },

        sectionCount() {
            try {
                let _tmp = ' - '
                if (this.filtredSections) {
                    if (this.filtredSections.length > 1)
                        _tmp += this.filtredSections.length + ' Sections'
                    else
                        _tmp += this.filtredSections.length + ' Section'

                } else {
                    _tmp += '0 Section'
                }
                return _tmp
            } catch (error) {
                console.error(error)
            }
        }

    },

    methods: {
        async init() {
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
        searchFilter(section, search) {
            try {
                if (section.tags) {
                    return section.tags.includes(search)
                }
                else if (section.name) {
                    return section.name.includes(search)
                } else
                    return false
            } catch (error) {
                console.error(error)
                return false
            }
        },


        async selectCategory(category) {
            this.selectedCategory = category;
            this.$forceUpdate();
        },

        async  selectSection(section) {
            console.log('store section section:', section)
            this.$emit('selectSection', section)
        },

        filter(section) {
            if (!section.preview || section.preview == 'aa' || section.preview == 'aaa' || !section.displayName) {
                return false;
            }
            if (!this.search) {
                return true;
            }

            if (section.displayName.en.toLowerCase().indexOf(this.search.toLowerCase()) !== -1) {
                return true;
            }
            return false;
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
        },    },
    mounted: function () {
        this.init()
    },
    beforeDestroy() {

    }
};
</script>

<style scoped lang="scss">
@import "../assets/manager-css/main";

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

.ww-popup-store {
    display: flex;
    flex-direction: column;
    height: 100%;
}

.content {
    width: 100%;
    display: flex;
    height: 100%;
    // overflow-y: hidden;
    flex-direction: column;

    .categories-container {
        display: flex;
        flex-direction: column;
        // sticky
        position: sticky;
        top: 0;
        padding-top: 50px;
        background-color: white;
        margin-bottom: 300px;
        z-index: 1;

        // flex-basis: calc(100% - 300px);
        .section-title {
            font-family: $ww-font;
            color: $ww-red-strong;
            text-transform: uppercase;
            font-weight: bold;
            font-size: 0.9rem;
            line-height: 13px;
            margin-bottom: 25px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding-right: 35px;

            .section-count {
                color: $ww-grey;
                margin-left: 5px;
                font-weight: normal;
                text-transform: capitalize;
                font-size: 1rem;
                line-height: 15px;
            }
            .search {
                display: flex;
                justify-content: center;
                height: fit-content;
                .search-bar {
                    width: 100%;
                }
            }
        }

        .filter-sections {
            display: flex;
            margin-bottom: 10px;
            justify-content: flex-start;
            align-items: center;
            padding-right: 35px;
            padding-left: 12px;

            .categories {
                flex-grow: 1;
                overflow-x: auto;
                padding-bottom: 10px;
                display: flex;
                justify-content: flex-start;
                flex-direction: row;
                flex-wrap: wrap;

                .category {
                    padding: 10px 20px 0 0;
                    font-family: $ww-font;
                    color: $ww-grey-light;
                    cursor: pointer;
                    transition: all 0.1s ease;
                    text-transform: capitalize;
                    font-size: 0.9rem;
                    line-height: 13px;
                    font-weight: 500;
                    &.active,
                    &:hover {
                        color: $ww-red;
                    }
                }
            }
        }
    }

    .sections {
        flex-grow: 1;
        overflow-y: auto;
        flex-wrap: wrap;
        font-family: $ww-font;
        display: flex;
        box-sizing: content-box;
        padding-right: 20px;
        justify-content: flex-start;
        align-content: flex-start;
        // sticky
        margin-top: -300px;
        .section {
            flex-basis: 33.3333%;
            padding: 0 12px;
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
</style>