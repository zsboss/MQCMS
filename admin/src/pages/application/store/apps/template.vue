<template>
    <Row :gutter="24" class="ivu-mt">
        <Col :xxl="4" :xl="6" :lg="6" :md="12" :sm="12" :xs="24" v-for="(item, index) in appList" :key="index" class="ivu-mb">
            <Card :shadow="!bordered" :bordered="bordered" :dis-hover="bordered" :padding="0">
                <div class="ivu-pt ivu-pl ivu-pr">
                    <div>
                        <Avatar :src="item.icon" />
                        <strong class="ivu-ml">{{ item.title }}</strong>
                    </div>
                    <div class="ivu-mt">
                        <div>负责人：{{ item.owner }}</div>
                        <div>最后更新时间：<Time :time="item.time" type="date" /></div>
                    </div>
                </div>
                <Divider class="ivu-mb-8" />
                <Row class="ivu-text-center ivu-pb-8">
                    <Col span="12" class="ivu-br">
                        <Tooltip placement="top" content="访问">
                            <i-link :to="item.link" target="_blank">
                                <Button icon="md-open" type="text" size="large" />
                            </i-link>
                        </Tooltip>
                    </Col>
                    <Col span="12">
                        <Tooltip placement="top" content="安装应用">
                            <i-link>
                                <Button icon="md-download" type="text" size="large" />
                            </i-link>
                        </Tooltip>
                    </Col>
                </Row>
            </Card>
        </Col>
    </Row>
</template>
<script>
    export default {
        props: {
            appList: {
                type: Array
            },
            compact: {
                type: Boolean,
                default: false
            },
            bordered: {
                type: Boolean,
                default: false
            }
        }
    }
</script>
