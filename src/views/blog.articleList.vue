/**
* Created by xiangsongtao on 16/7/24.
* Description: 文章列表
*/
<template>
    <div class="aritcleList" transition="blogTrans">
        <article class="article animated fadeIn" v-for="article of articleList" v-link="{ name: 'article',params: { articleId: article._id },activeClass: 'active'}">
            <div class="article__header">
                <h2 class="article__header--title">{{article.title}}</h2>
                <div class="article__header--content">
                    {{article.content}}
                </div>
            </div>
            <div class="article__infobox">
                <div class="article__info">
                    <div class="article__info--each">
                        <i class="fa fa-calendar"></i>
                        <span>{{article.publish_time  | moment "from" "now"}}</span>
                    </div>
                    <div class="article__info--each">
                        <i class="fa fa-book"></i>
                        阅读数
                        <span class="article-readnum">{{article.read_num}}</span>
                    </div>
                    <div class="article__info--each">
                        <i class="fa fa-comments"></i>评论数
                        <span class="article-comment">{{article.comment_num}}</span>
                    </div>
                    <div class="article__info--each hidden-xs" v-for="tag of article.tags">
                        <i class="fa fa-tag"></i> <span>{{tag.name}}</span>
                    </div>
                </div>
                <div class="arrticle__readmore hidden-xs">
                    <span>阅读更多</span>
                </div>
            </div>
        </article>
        <section class="copyright animated fadeIn" v-if="articleList.length!==0">
            <copyright></copyright>
        </section>
        <no-data v-if="!hasData && !isLoading"></no-data>
        <is-loading v-if="isLoading"></is-loading>
    </div>
</template>
<style scoped lang="scss">
    @import "../theme/theme.scss";

    /*内容区*/
    .aritcleList {
        width: 780px;
        margin: 0 auto;
        /*position: relative;*/

        .article {
            user-select: none;
            -webkit-user-select: none;
            box-sizing: border-box;
            margin-bottom: 30px;
            cursor: pointer;
            /*border-radius: 4px;*/
            overflow: hidden;

            &:hover .article__header .article__header--title {
                color: $base-theme-color;
                &:after {
                    border-top: 3px solid $base-theme-color;
                }
            }
            &:hover .article__infobox .arrticle__readmore span {
                background-color: $base-theme-color;
            }
            //标题
            .article__header {
                padding: 35px;
                color: $base-word-color;
                font-size: 20px;
                line-height: 28px;
                background: #fff;
                position: relative;
                //title
                .article__header--title {
                    text-align: right;
                    transition: color ease 200ms;
                    position: relative;
                    padding-bottom: 20px;
                    margin: 0;
                    font-size: 30px;
                    font-weight: 500;
                    line-height: 1.2;
                    &:after {
                        transition: color ease 200ms;
                        content: '';
                        position: absolute;
                        bottom: 10px;
                        right: 0;
                        height: 0;
                        border-top: 3px solid $base-theme-color;
                        border-top-right-radius: 3px;
                        border-bottom-right-radius: 3px;

                        //background: #333;
                        width: 10%;
                    }
                }
                //内容
                .article__header--content {
                    line-height: 24px;
                    font-size: 16px;
                    overflow: hidden;
                    p {
                        margin: 0;
                    }
                }
            }
            //文章信息
            .article__infobox {
                @include display-flex;
                @include justify-content(space-between);
                @include align-items(center);
                background: $base-background-color;
                padding: 14px 35px;
                box-shadow: 0 2px 2px 0 rgba(0, 0, 0, 0.8) inset;
                transition: all ease 200ms;
                .article__info {
                    @include display-flex;
                    @include justify-content(flex-start);
                    @include align-items(center);
                    .article__info--each {
                        color: #eee;
                        margin-right: 20px;
                        font-size: 14px;
                        white-space: nowrap;
                    }
                }
                //阅读更多
                .arrticle__readmore {
                    span {
                        border: 1px solid $base-theme-color;
                        transition: all ease 200ms;
                        border-radius: 34px;
                        color: #eee;
                        padding: 4px 22px;
                        font-size: 16px;
                    }

                }
            }

        }
    }

    @include media("<=tablet") {
        .aritcleList {
            max-width: 780px;
            width: 100%;
            .article {
                .article__header {
                    padding: 30px;
                }
                .article__infobox {
                    padding: 14px 20px
                }
            }
        }
    }

    @include media("<=phone") {
        .aritcleList {
            max-width: 780px;
            width: 100%;
            .article {
                .article__header {
                    padding: 30px 10px 10px;
                    .article__header--title {
                        font-size: 28px;
                    }
                    .article__header--content {
                        font-size: 14px;
                    }
                }
                .article__infobox {
                    padding: 10px 0px;
                    display: flex;
                    justify-content: flex-end;
                    align-items:center;
                    .article__info {
                        display: flex;
                        justify-content: space-between;
                        align-items: center;
                        .article__info--each {
                            font-size: 14px;
                            margin: 0 10px;
                        }
                    }
                }
            }
        }
    }

</style>
<script>
    import API from "../config.js"
    import noData from "../components/nodata.vue"
    import isLoading from "../components/isLoading.vue"
    import copyright from '../components/copyright.vue'
    import {GetArticleListForFrontEnd} from "../api/api_article"

    export default{
        replace: true,
        data: function () {
            return {
                isLoading: true,
                articleList: [],
                hasData: true,
            }
        },
        methods: {},
        ready: function () {
            const scope = this;
            $(window).scrollTop(0);// 滚到顶部
            /**
             * 文章列表会根据
             * "最新-latest"、"标签筛选-tagList"进行区分,
             * 不同的type进行不同的url搜索
             * */
            let listType = scope.$route.query.listType;
            let url;
            switch (listType) {
                case 'latest':
                    url = API.newUpdateArticle.replace("from", API.ArticleFrom).replace("to", API.ArticleTo);
                    break;
                case 'tagList':
                    url = API.getArticlesWithTagId.replace('id', scope.$route.query.tagId);
                    break;
            }
            // 数据请求根据url参数会发生改变,故get时传入url
            GetArticleListForFrontEnd(url).then((data)=> {
                scope.articleList = data;
            }, ()=> {
                scope.hasData = false;
            }).then(function () {
                scope.articleList.length === 0 ? (scope.hasData = false) : (scope.hasData = true)
                scope.isLoading = false;
            })
        },
        destroyed: function () {
        },
        components: {
            noData, copyright, isLoading,
        },
    }
</script>
