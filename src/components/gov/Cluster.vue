<template>
    <div>
        <el-collapse v-model="activeNames" style="margin-left: 170px; width: 88%">
            <el-collapse-item title="Pods列表" name="1">
                <el-table :data="pods" :stripe="true" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="Pod名称" width="180px"></el-table-column> 
                    <el-table-column prop="metadata.namespace" label="命名空间" width="180px"></el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述" width="360px"></el-table-column>
                    <el-table-column prop="spec.containers[0].image" label="镜像"></el-table-column>
                </el-table>
            </el-collapse-item>

            <el-collapse-item title="Deployments列表" name="2">
                <el-table :data="deployments" :stripe="true" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="Deployment名称" width="135px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="90px"></el-table-column>
                    <el-table-column prop="spec.replicas" label="负载数量" width="90px"></el-table-column>
                    <el-table-column prop="spec.selector.matchLabels.app" label="app标签" width="90px"></el-table-column>
                    <el-table-column prop="spec.selector.matchLabels.version" label="version标签" width="120px"></el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述" width="390px"></el-table-column>
                    <el-table-column prop="spec.template.spec.containers[0].image" label="镜像" ></el-table-column>
                </el-table>
            </el-collapse-item>

            <el-collapse-item title="StatefulSets列表" name="3">
                <el-table :data="statefulsets" :stripe="true" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="StatefulSet名称" width="135px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="90px"></el-table-column>
                    <el-table-column prop="spec.replicas" label="负载数量" width="90px"></el-table-column>
                    <el-table-column prop="spec.selector.matchLabels.app" label="app标签" width="90px"></el-table-column>
                    <el-table-column prop="spec.selector.matchLabels.version" label="version标签" width="120px"></el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述" width="390px"></el-table-column>
                    <el-table-column prop="spec.template.spec.containers[0].image" label="镜像" ></el-table-column>
                </el-table>
            </el-collapse-item>

            <el-collapse-item title="Services列表" name="4">
                <el-table :data="services" :stripe="true" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="Service名称" width="180px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="180px"></el-table-column>
                    <el-table-column prop="spec.selector.app" label="app标签" width="180px"></el-table-column>
                    <el-table-column prop="spec.type" label="类型" width="180px"></el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述"></el-table-column>
                </el-table>
            </el-collapse-item>

            <el-collapse-item title="Gateway信息" name="5">
                <el-table :data="gateway" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="Gateway名称" width="180px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="180px"></el-table-column>
                    <el-table-column label="网关规则" width="240px">
                        <el-button type="primary" plain size="small" @click="gatewayStrVisible = true" icon="el-icon-tickets">查看详情</el-button>
                    </el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述"></el-table-column>
                </el-table>
                <el-dialog title="网关规则" :visible.sync="gatewayStrVisible" width="60%" height="30%" :before-close="handleDialogClose">
                    <el-input v-model="gatewayStr" autosize type="textarea"></el-input>
                </el-dialog>
            </el-collapse-item>

            <el-collapse-item title="VirtualService信息" name="6">
                <el-table :data="virtualService" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="VS名称" width="180px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="180px"></el-table-column>
                    <el-table-column label="流量调度规则" width="240px">
                        <el-button type="primary" plain size="small" @click="virtualServiceDialogVisible = true" icon="el-icon-tickets">查看详情</el-button>
                        <el-button type="primary" plain size="small" @click="handleVirtualServiceConfigOpen" icon="el-icon-edit-outline">负载均衡</el-button>
                    </el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述"></el-table-column>
                </el-table>
                <el-dialog title="VirtualService流量调度规则" :visible.sync="virtualServiceDialogVisible" width="60%" height="30%" :before-close="handleDialogClose">
                    <el-input v-model="virtualServiceStr" autosize type="textarea"></el-input>
                </el-dialog>
                <el-dialog title="VirtualService负载权重配置" :visible.sync="virtualServiceConfigVisible" width="60%" height="30%" :before-close="handleDialogClose">
                    <el-button type="primary" plain size="small" @click="applyVirtualService" icon="el-icon-document-copy" style="float: left; margin-bottom: 5px;">保存配置</el-button>
                    <el-button type="primary" plain size="small" @click="applyTemplateVirtualService" icon="el-icon-reading" style="float: left; margin-left:10px; margin-bottom: 5px;">一键生效模板配置</el-button>
                    <el-table :data="virtualServiceConfig" max-height="250" :border="true">
                        <el-table-column prop="destination.host" label="Host信息" width="180px"></el-table-column>
                        <el-table-column prop="destination.port.number" label="端口" width="180px"></el-table-column>
                        <el-table-column prop="destination.subset" label="子集" width="180px"></el-table-column>
                        <el-table-column prop="weight" label="权重">
                            <template slot-scope="scope">
                                <el-input v-model="scope.row.weight" placeholder="请输入权重比例"></el-input>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-dialog>
            </el-collapse-item>

            <el-collapse-item title="DestinationRule信息" name="7">
                <el-table :data="destinationRule" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="DR名称" width="180px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="180px"></el-table-column>
                    <el-table-column label="目标规则配置" width="240px">
                        <el-button type="primary" plain size="small" @click="handleDestinationRuleConfigOpen" icon="el-icon-tickets">查看详细配置</el-button>
                    </el-table-column>
                    <el-table-column prop="metadata.selfLink" label="完整描述"></el-table-column>
                </el-table>
                <el-dialog title="DestinationRule配置详情" :visible.sync="destinationRuleDialogVisible" width="60%" :before-close="handleDialogClose">
                        <el-button type="primary" plain size="small" @click="destinationRuleDemoteDialogVisible = true" icon="el-icon-tickets" style="float: left; margin-bottom: 5px;">查看完整配置</el-button>
                        <el-dialog title="DestinationRule完整配置" :visible.sync="destinationRuleDemoteDialogVisible" append-to-body :before-close="handleDialogClose">
                            <el-input v-model="destinationRuleStr" autosize type="textarea"></el-input>
                        </el-dialog>
                        <el-button type="primary" plain size="small" @click="handleDestinationRuleGlobalLoadBalancerConfigOpen" icon="el-icon-edit-outline" style="float: left; margin-left:10px; margin-bottom: 5px;">全局负载均衡策略配置</el-button>
                        <el-dialog title="全局负载均衡策略配置" :visible.sync="destinationRuleGlobalLoadBalancerVisible" append-to-body :before-close="handleDialogClose">
                            <el-radio v-model="destinationRuleGlobalLoadBalancerConfig" :label="0">关闭</el-radio>
                            <el-radio v-model="destinationRuleGlobalLoadBalancerConfig" :label="1">ROUND_ROBIN</el-radio>
                            <el-radio v-model="destinationRuleGlobalLoadBalancerConfig" :label="2">LEAST_CONN</el-radio>
                            <el-radio v-model="destinationRuleGlobalLoadBalancerConfig" :label="3">RANDOM</el-radio>
                            <el-radio v-model="destinationRuleGlobalLoadBalancerConfig" :label="4">PASSTHROUGH</el-radio>
                            <el-button type="primary" plain size="small" @click="applyDestinationRuleGlobalLoadBalancer">生效</el-button>
                        </el-dialog>
                        <el-button type="primary" plain size="small" @click="applyTemplateDestinationRule" icon="el-icon-reading" style="float: left; margin-left:10px; margin-bottom: 5px;">一键生效模板配置</el-button>
                        <el-table :data="destinationRuleConfig" :border="true">
                            <el-table-column prop="name" label="子集名称" width="180px"></el-table-column>
                            <el-table-column prop="labels.version" label="version标签" width="180px"></el-table-column>
                            <el-table-column label="">
                                <template slot-scope="scope">
                                    <el-button type="primary" plain size="small" @click="handleDestinationRuleDemoteSubsetOverrideLoadBalancerOpen(scope.$index)" icon="el-icon-document-copy" style="float: left;">覆盖负载策略</el-button>
                                    <el-dialog title="覆盖负载均衡策略配置" :visible.sync="destinationRuleDemoteSubsetLoadBalancerVisible" append-to-body :before-close="handleDialogClose">
                                        <el-radio v-model="destinationRuleDemoteSubsetLoadBalancerConfig" :label="0">关闭</el-radio>
                                        <el-radio v-model="destinationRuleDemoteSubsetLoadBalancerConfig" :label="1">ROUND_ROBIN</el-radio>
                                        <el-radio v-model="destinationRuleDemoteSubsetLoadBalancerConfig" :label="2">LEAST_CONN</el-radio>
                                        <el-radio v-model="destinationRuleDemoteSubsetLoadBalancerConfig" :label="3">RANDOM</el-radio>
                                        <el-radio v-model="destinationRuleDemoteSubsetLoadBalancerConfig" :label="4">PASSTHROUGH</el-radio>
                                        <el-button type="primary" plain size="small" @click="applyDestinationRuleOverrideLoadBalancer()">生效</el-button>
                                    </el-dialog>
                                    <el-button type="primary" plain size="small" v-if="scope.row.state" @click="handleDestinationRuleDemoteConfigOpen(scope.$index)" icon="el-icon-edit-outline" style="float: left; margin-left:10px">熔断配置</el-button>
                                    <el-button type="primary" plain size="small" v-if="!scope.row.state" @click="handleDestinationRuleDemoteConfigOpen(scope.$index)" icon="el-icon-edit" style="float: left; margin-left:10px">开启熔断</el-button>
                                    <el-dialog title="熔断配置" :visible.sync="destinationRuleDemoteConfigVisible" append-to-body :before-close="handleDialogClose">
                                        <el-form :ref="destinationRuleDemoteConfigForm" :model="destinationRuleDemoteConfigForm" label-position="left" label-width="210px">
                                            <el-form-item label="子集名称">{{ destinationRuleDemoteConfigForm.name }}</el-form-item>
                                            <el-form-item label="version标签">{{ destinationRuleDemoteConfigForm.labels.version }}</el-form-item>
                                            <el-form-item label="tcp最大连接数">
                                                <el-input v-model="destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.maxConnections" style="width: 80%"></el-input>
                                            </el-form-item>
                                            <el-form-item label="tcp连接超时时间">
                                                <el-input v-model="destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.connectTimeout" style="width: 80%"></el-input>
                                            </el-form-item>
                                            <el-form-item label="http1最大排队请求数">
                                                <el-input v-model="destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.http1MaxPendingRequests" style="width: 80%"></el-input>
                                            </el-form-item>
                                            <el-form-item label="http单个连接最大请求数量">
                                                <el-input v-model="destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.maxRequestsPerConnection" style="width: 80%"></el-input>
                                            </el-form-item>
                                            <el-form-item label="熔断检测中连续5xx错误码数量">
                                                <el-input v-model="destinationRuleDemoteConfigForm.trafficPolicy.outlierDetection.consecutive5xxErrors" style="width: 80%"></el-input>
                                            </el-form-item>
                                        </el-form>
                                        <el-button type="primary" plain size="small" @click="applyDestinationRuleDemote" icon="el-icon-document-checked">生效</el-button>
                                    </el-dialog>
                                    <el-button type="primary" plain size="small" v-if="scope.row.state" @click="handleDestinationRuleDemoteConfigOfSubsetDelete(scope.$index)" icon="el-icon-delete" style="float: left; margin-left:10px">关闭熔断</el-button>
                                    <el-button type="primary" plain size="small" @click="handleDestinationRuleDemoteSubsetOpen(scope.$index)" icon="el-icon-tickets" style="float:left; margin-left:10px">查看完整配置</el-button>
                                    <el-dialog title="完整配置" :visible.sync="destinationRuleDemoteSubsetDialogVisible" append-to-body :before-close="handleDialogClose">
                                        <el-input v-model="destinationRuleDemoteSubsetStr" autosize type="textarea"></el-input>
                                    </el-dialog>
                                </template>
                            </el-table-column>
                        </el-table>
                </el-dialog>
            </el-collapse-item>

            <el-collapse-item title="限流配置列表" name="8">
                <el-button type="primary" plain size="small" @click="handleEnvoyFilterNewConfigOpen" icon="el-icon-edit" style="float: left; margin-bottom: 5px;">新建限流配置</el-button>
                <el-dialog title="新建限流服务配置" :visible.sync="envoyFilterNewConfigVisible" width="60%" height="30%" :before-close="handleDialogClose">
                                <el-form :ref="envoyFilterConfigForm" :model="envoyFilterConfigForm" label-position="left" label-width="150px">
                                    <el-form-item label="限流服务名称">
                                        <el-input v-model="envoyFilterConfigForm.metadata.name" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="限流对象app标签">
                                        <el-input v-model="envoyFilterConfigForm.spec.workloadSelector.labels.app" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="限流对象version标签">
                                        <el-input v-model="envoyFilterConfigForm.spec.workloadSelector.labels.version" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="令牌数量">
                                        <el-input v-model="envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="每次填充令牌数">
                                        <el-input v-model="envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill" style="width: 80%"></el-input>
                                    </el-form-item>
                                </el-form>
                                <el-button type="primary" plain size="small" @click="applyEnvoyFilter" icon="el-icon-document-checked">创建</el-button>
                            </el-dialog>
                <el-table :data="envoyFilters" max-height="250" :border="true">
                    <el-table-column prop="metadata.name" label="限流服务名称" width="180px"></el-table-column>
                    <el-table-column prop="metadata.namespace" label="命名空间" width="120px"></el-table-column>
                    <el-table-column prop="spec.workloadSelector.labels.app" label="生效服务app" width="120px"></el-table-column>
                    <el-table-column prop="spec.workloadSelector.labels.version" label="生效服务version" width="150px"></el-table-column>
                    <el-table-column prop="spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens" label="令牌数量" width="120px"></el-table-column>
                    <el-table-column prop="spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill" label="每次填充令牌数" width="150px"></el-table-column>
                    <el-table-column label="操作">
                        <template slot-scope="scope">
                            <el-button type="primary" plain size="small" @click="handleEnvoyFilterDialogOpen(scope.$index)"  icon="el-icon-tickets" style="float: left;">查看完整配置</el-button>
                            <el-dialog title="限流服务完整配置" :visible.sync="envoyFilterDialogVisible" width="60%" height="30%" :before-close="handleDialogClose">
                                <el-input v-model="envoyFilterStr" autosize type="textarea"></el-input>
                            </el-dialog>
                            <el-button type="primary" plain size="small" @click="handleEnvoyFilterConfigOpen(scope.$index)" icon="el-icon-edit-outline" style="float: left; margin-left:10px;">编辑配置</el-button>
                            <el-dialog title="限流服务配置编辑" :visible.sync="envoyFilterConfigVisible" width="60%" height="30%" :before-close="handleDialogClose">
                                <el-form :ref="envoyFilterConfigForm" :model="envoyFilterConfigForm" label-position="left" label-width="210px">
                                    <el-form-item label="限流服务名称">
                                        <el-input v-model="envoyFilterConfigForm.metadata.name" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="限流对象app标签">
                                        <el-input v-model="envoyFilterConfigForm.spec.workloadSelector.labels.app" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="限流对象version标签">
                                        <el-input v-model="envoyFilterConfigForm.spec.workloadSelector.labels.version" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="令牌数量">
                                        <el-input v-model="envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens" style="width: 80%"></el-input>
                                    </el-form-item>
                                    <el-form-item label="每次填充令牌数">
                                        <el-input v-model="envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill" style="width: 80%"></el-input>
                                    </el-form-item>
                                </el-form>
                                <el-button type="primary" plain size="small" @click="applyEnvoyFilter" icon="el-icon-document-checked">生效</el-button>
                            </el-dialog>
                            <el-button type="primary" plain size="small" @click="deleteEnvoyFilter(scope.$index)" icon="el-icon-delete" style="float: left; margin-left:10px;">删除</el-button>
                        </template>
                    </el-table-column>
                    <!-- <el-table-column prop="metadata.selfLink" label="完整描述"></el-table-column> -->
                </el-table>
            </el-collapse-item>
        </el-collapse>
    </div>
</template>

<script>
    export default {
        name: 'Cluster',
        data () {
            return {
                clusterId: 1,
                activeNames: ['6', '7', '8'],
                pods: [
                    {
                        "metadata": {
                            "name": "mysql1-0",
                            "namespace": "default",
                            "selfLink": "/api/v1/namespaces/default/pods/mysql1-0",
                        },
                        "spec": {
                            "containers": [
                                {
                                    "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                }
                            ]
                        }
                    },
                    {
                        "metadata": {
                            "name": "mysql2-0",
                            "namespace": "default",
                            "selfLink": "/api/v1/namespaces/default/pods/mysql2-0",
                        },
                        "spec": {
                            "containers": [
                                {
                                    "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                }
                            ]
                        }
                    },
                    {
                        "metadata": {
                            "name": "mysql3-0",
                            "namespace": "default",
                            "selfLink": "/api/v1/namespaces/default/pods/mysql3-0",
                        },
                        "spec": {
                            "containers": [
                                {
                                    "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                }
                            ]
                        }
                    }
                ],
                deployments: [
                    {
                        "metadata": {
                            "name": "server-v1",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/deployments/server-v1"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "server",
                                    "version": "v1"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/music-server:v1"
                                        }
                                    ]
                                }
                            }
                        }
                    },
                    {
                        "metadata": {
                            "name": "server-v2",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/deployments/server-v2"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "server",
                                    "version": "v2"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/music-server:v2"
                                        }
                                    ]
                                }
                            }
                        }
                    },
                    {
                        "metadata": {
                            "name": "server-v3",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/deployments/server-v3"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "server",
                                    "version": "v3"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/music-server:v1"
                                        }
                                    ]
                                }
                            }
                        }
                    }
                ],
                statefulsets: [
                    {
                        "metadata": {
                            "name": "mysql1",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/statefulsets/mysql1"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "mysql",
                                    "version": "v1"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                        }
                                    ]
                                }
                            }
                        }
                    },
                    {
                        "metadata": {
                            "name": "mysql2",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/statefulsets/mysql2"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "mysql",
                                    "version": "v2"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                        }
                                    ]
                                }
                            }
                        }
                    },
                    {
                        "metadata": {
                            "name": "mysql3",
                            "namespace": "default",
                            "selfLink": "/apis/apps/v1/namespaces/default/statefulsets/mysql3"
                        },
                        "spec": {
                            "replicas": 1,
                            "selector": {
                                "matchLabels": {
                                    "app": "mysql",
                                    "version": "v3"
                                }
                            },
                            "template": {
                                "spec": {
                                    "containers": [
                                        {
                                            "image": "registry.cn-hangzhou.aliyuncs.com/skeet/mysql:8.0.30"
                                        }
                                    ]
                                }
                            }
                        }
                    }
                ],
                services: [
                    {
                        "metadata": {
                            "name": "server",
                            "namespace": "default",
                            "selfLink": "/api/v1/namespaces/default/services/server"
                        },
                        "spec": {
                            "selector": {
                                "app": "server"
                            },
                            "type": "ClusterIP"
                        }
                    },
                    {
                        "metadata": {
                            "name": "mysql",
                            "namespace": "default",
                            "selfLink": "/api/v1/namespaces/default/services/mysql"
                        },
                        "spec": {
                            "selector": {
                                "app": "mysql"
                            },
                            "type": "ClusterIP"
                        }
                    }
                ],
                gateway: [
                    {
                        "metadata": {
                            "name": "server-gateway",
                            "namespace": "default",
                            "selfLink": "/apis/networking.istio.io/v1alpha3/namespaces/default/gateways/server-gateway"
                        },
                        "spec": {
                            "servers": [
                                {
                                    "hosts": [
                                        "*"
                                    ],
                                    "port": {
                                        "name": "http",
                                        "number": 80,
                                        "protocol": "HTTP"
                                    }
                                },
                                {
                                    "hosts": [
                                        "*"
                                    ],
                                    "port": {
                                        "name": "https",
                                        "number": 443,
                                        "protocol": "HTTP"
                                    }
                                }
                            ]
                        }
                    }
                ],
                gatewayStr: '',
                gatewayStrVisible: false,
                virtualService: [
                    {
                        "metadata": {
                            "name": "server",
                            "namespace": "default",
                            "selfLink": "/apis/networking.istio.io/v1alpha3/namespaces/default/virtualservices/server",
                        },
                        "spec": {
                            "gateways": [
                                "server-gateway"
                            ],
                            "hosts": [
                                "*"
                            ],
                            "http": [
                                {
                                    "match": [
                                        {
                                            "port": 443
                                        }
                                    ],
                                    "route": [
                                        {
                                            "destination": {
                                                "host": "server",
                                                "port": {
                                                    "number": 8088
                                                },
                                                "subset": "v1"
                                            },
                                            "weight": 40
                                        },
                                        {
                                            "destination": {
                                                "host": "server",
                                                "port": {
                                                    "number": 8088
                                                },
                                                "subset": "v2"
                                            },
                                            "weight": 20
                                        },
                                        {
                                            "destination": {
                                                "host": "server",
                                                "port": {
                                                    "number": 8088
                                                },
                                                "subset": "v3"
                                            },
                                            "weight": 40
                                        }
                                    ]
                                }
                            ]
                        }
                    }
                ],
                virtualServiceConfig: [
                    {
                        "destination": {
                            "host": "server",
                            "port": {
                                "number": 8088
                            },
                            "subset": "v1"
                        },
                        "weight": 40
                    },
                    {
                        "destination": {
                            "host": "server",
                            "port": {
                                "number": 8088
                            },
                            "subset": "v2"
                        },
                        "weight": 20
                    },
                    {
                        "destination": {
                            "host": "server",
                            "port": {
                                "number": 8088
                            },
                            "subset": "v3"
                        },
                        "weight": 40
                    }
                ],
                virtualServiceStr: '',
                virtualServiceDialogVisible: false,
                virtualServiceConfigVisible: false,
                destinationRule: [
                    {
                        "metadata": {
                            "name": "server",
                            "namespace": "default",
                            "selfLink": "/apis/networking.istio.io/v1alpha3/namespaces/default/destinationrules/server",
                        },
                        "spec": {
                            "host": "server",
                            "subsets": [
                                {
                                    "labels": {
                                        "version": "v1"
                                    },
                                    "name": "v1"
                                },
                                {
                                    "labels": {
                                        "version": "v2"
                                    },
                                    "name": "v2",
                                    "trafficPolicy": {
                                        "loadBalancer": {
                                            "simple": "RANDOM"
                                        }
                                    }
                                },
                                {
                                    "labels": {
                                        "version": "v3"
                                    },
                                    "name": "v3",
                                    "trafficPolicy": {
                                        "connectionPool": {
                                            "http": {
                                                "http1MaxPendingRequests": 1,
                                                "maxRequestsPerConnection": 1
                                            },
                                            "tcp": {
                                                "connectTimeout": "30ms",
                                                "maxConnections": 1,
                                                "tcpKeepalive": {
                                                    "interval": "75s",
                                                    "time": "7200s"
                                                }
                                            }
                                        },
                                        "outlierDetection": {
                                            "baseEjectionTime": "3m",
                                            "consecutive5xxErrors": 1,
                                            "interval": "1s",
                                            "maxEjectionPercent": 100
                                        },
                                        "loadBalancer": {
                                            "simple": "RANDOM"
                                        }
                                    }
                                }
                            ],
                            "trafficPolicy": {
                                "loadBalancer": {
                                    "simple": "ROUND_ROBIN"
                                }
                            }
                        }
                    }
                ],
                destinationRuleConfig: [
                    {
                        "labels": {
                            "version": "v1"
                        },
                        "name": "v1",
                        "state": false,
                        "trafficPolicy": {
                            "connectionPool": {
                                "http": {
                                    "http1MaxPendingRequests": 1,
                                    "maxRequestsPerConnection": 1
                                },
                                "tcp": {
                                    "connectTimeout": "30ms",
                                    "maxConnections": 1,
                                    "tcpKeepalive": {
                                        "interval": "75s",
                                        "time": "7200s"
                                    }
                                }
                            },
                            "outlierDetection": {
                                "baseEjectionTime": "3m",
                                "consecutive5xxErrors": 1,
                                "interval": "1s",
                                "maxEjectionPercent": 100
                            },
                            "loadBalancer": {
                                "simple": "RANDOM"
                            }
                        }
                    },
                    {
                        "labels": {
                            "version": "v2"
                        },
                        "name": "v2",
                        "state": false,
                        "trafficPolicy": {
                            "connectionPool": {
                                "http": {
                                    "http1MaxPendingRequests": 1,
                                    "maxRequestsPerConnection": 1
                                },
                                "tcp": {
                                    "connectTimeout": "30ms",
                                    "maxConnections": 1,
                                    "tcpKeepalive": {
                                        "interval": "75s",
                                        "time": "7200s"
                                    }
                                }
                            },
                            "outlierDetection": {
                                "baseEjectionTime": "3m",
                                "consecutive5xxErrors": 1,
                                "interval": "1s",
                                "maxEjectionPercent": 100
                            },
                            "loadBalancer": {
                                "simple": "RANDOM"
                            }
                        }
                    },
                    {
                        "labels": {
                            "version": "v3"
                        },
                        "name": "v3",
                        "state": true,
                        "trafficPolicy": {
                            "connectionPool": {
                                "http": {
                                    "http1MaxPendingRequests": 1,
                                    "maxRequestsPerConnection": 1
                                },
                                "tcp": {
                                    "connectTimeout": "30ms",
                                    "maxConnections": 1,
                                    "tcpKeepalive": {
                                        "interval": "75s",
                                        "time": "7200s"
                                    }
                                }
                            },
                            "outlierDetection": {
                                "baseEjectionTime": "3m",
                                "consecutive5xxErrors": 1,
                                "interval": "1s",
                                "maxEjectionPercent": 100
                            },
                            "loadBalancer": {
                                "simple": "RANDOM"
                            }
                        }
                    }
                ],
                destinationRuleStr: '',
                destinationRuleGlobalLoadBalancerVisible: false,
                destinationRuleGlobalLoadBalancer: '',
                destinationRuleGlobalLoadBalancerConfig: '',
                destinationRuleDialogVisible: false,
                destinationRuleDemoteDialogVisible: false,
                destinationRuleDemoteConfigVisible: false,
                destinationRuleDemoteConfigForm: {
                    "labels": {
                        "version": ''
                    },
                    "name": '',
                    "state": '',
                    "trafficPolicy": {
                        "connectionPool": {
                            "http": {
                                "http1MaxPendingRequests": '',
                                "maxRequestsPerConnection": ''
                            },
                            "tcp": {
                                "connectTimeout": '',
                                "maxConnections": '',
                                "tcpKeepalive": {
                                    "interval": "75s",
                                    "time": "7200s"
                                }
                            }
                        },
                        "outlierDetection": {
                            "baseEjectionTime": "3m",
                            "consecutive5xxErrors": 1,
                            "interval": "1s",
                            "maxEjectionPercent": 100
                        },
                        "loadBalancer": {
                            "simple": ''
                        }
                    }
                },
                destinationRuleDemoteSubsetDialogVisible: false,
                destinationRuleDemoteSubsetStr: '',
                destinationRuleDemoteSubsetLoadBalancerVisible: false,
                destinationRuleDemoteSubsetLoadBalancerConfig: '',
                destinationRuleCurrentSubsetIndex: '',
                loadBalancerPolicy: ['', 'ROUND_ROBIN', 'LEAST_CONN', 'RANDOM', "PASSTHROUGH"],
                envoyFilters: [
                    {
                        "metadata": {
                            "name": "filter-local-ratelimit-svc",
                            "namespace": "istio-system",
                            "selfLink": "/apis/networking.istio.io/v1alpha3/namespaces/istio-system/envoyfilters/filter-local-ratelimit-svc"
                        },
                        "spec": {
                            "configPatches": [
                                {
                                    "patch": {
                                        "value": {
                                            "typed_config": {
                                                "value": {
                                                    "token_bucket": {
                                                        "fill_interval": "60s",
                                                        "max_tokens": 5,
                                                        "tokens_per_fill": 10
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            ],
                            "workloadSelector": {
                                "labels": {
                                    "app": "server"
                                }
                            }
                        }
                    },
                    {
                        "metadata": {
                            "name": "filter-local-ratelimit-svc2",
                            "namespace": "istio-system",
                            "selfLink": "/apis/networking.istio.io/v1alpha3/namespaces/istio-system/envoyfilters/filter-local-ratelimit-svc"
                        },
                        "spec": {
                            "configPatches": [
                                {
                                    "patch": {
                                        "value": {
                                            "typed_config": {
                                                "value": {
                                                    "token_bucket": {
                                                        "fill_interval": "60s",
                                                        "max_tokens": 10,
                                                        "tokens_per_fill": 10
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                            ],
                            "workloadSelector": {
                                "labels": {
                                    "app": "server",
                                    "version": "v2"
                                }
                            }
                        }
                    }
                ],
                envoyFilterDialogVisible: false,
                envoyFilterStr: '',
                envoyFilterNewConfigVisible: false,
                envoyFilterConfigVisible: false,
                envoyFilterCurrentIndex: '',
                envoyFilterConfigForm: {
                    "metadata": {
                        "name": '',
                        "namespace": 'istio-system',
                        "selfLink": ''
                    },
                    "spec": {
                        "configPatches": [
                            {
                                "patch": {
                                    "value": {
                                        "typed_config": {
                                            "value": {
                                                "token_bucket": {
                                                    "fill_interval": "60s",
                                                    "max_tokens": '',
                                                    "tokens_per_fill": ''
                                                }
                                            }
                                        }
                                    }
                                }
                            }
                        ],
                        "workloadSelector": {
                            "labels": {
                                "app": '',
                                "version": ''
                            }
                        }
                    }
                }
            }
        },
        mounted: function() {
            this.init()
        },
        watch: {
            '$route' (to, from) {
                this.init()
            }
        },
        methods: {
            init() {
                this.clusterId = this.$route.query.clusterId
                this.getPods()
                this.getDeployments()
                this.getStatefulSets()
                this.getServices()
                this.getGateway()
                this.getVirtualService()
                this.getDestinationRule()
                this.getEnvoyFilters()
                this.gatewayStr = JSON.stringify(this.gateway[0].spec, null, '\t')
                this.virtualServiceStr = JSON.stringify(this.virtualService[0].spec, null, '\t')
                this.destinationRuleStr = JSON.stringify(this.destinationRule[0].spec, null, '\t')
            },
            handleDialogClose(done) {
                done();
            },
            handleConfigClose(done) {
                this.$confirm('确认关闭？')
                    .then(_ => {
                        done();
                    })
                    .catch(_ => {});
            },
            handleVirtualServiceConfigOpen() {
                this.virtualServiceConfigVisible = true
                this.virtualServiceConfig = JSON.parse(JSON.stringify(this.virtualService[0].spec.http[0].route))
            },
            handleDestinationRuleConfigOpen() {
                this.destinationRuleDialogVisible = true
                for (let i = 0; i < this.destinationRule[0].spec.subsets.length; ++i) {
                    this.destinationRuleConfig[i] = JSON.parse(JSON.stringify(this.destinationRule[0].spec.subsets[i]))
                    if (this.destinationRule[0].spec.subsets[i].trafficPolicy && this.destinationRule[0].spec.subsets[i].trafficPolicy.connectionPool) {
                        this.destinationRuleConfig[i].state = true
                    } else {
                        this.destinationRuleConfig[i].state = false
                    }
                }
                if (this.destinationRule[0].spec.trafficPolicy) {
                    var currentPolicy = this.destinationRule[0].spec.trafficPolicy.loadBalancer.simple
                    for (let i = 0; i < this.loadBalancerPolicy.length; ++i) {
                        if (this.loadBalancerPolicy[i] === currentPolicy) {
                            this.destinationRuleGlobalLoadBalancer = i
                        }
                    }
                } else {
                    this.destinationRuleGlobalLoadBalancer = 0
                }
            },
            handleDestinationRuleGlobalLoadBalancerConfigOpen( ) {
                this.destinationRuleGlobalLoadBalancerVisible = true
                this.destinationRuleGlobalLoadBalancerConfig = this.destinationRuleGlobalLoadBalancer
            },
            handleDestinationRuleDemoteSubsetOpen(index) {
                this.destinationRuleDemoteSubsetDialogVisible = true
                this.destinationRuleDemoteSubsetStr = JSON.stringify(this.destinationRuleConfig[index])
            },
            handleDestinationRuleDemoteConfigOfSubsetDelete(index) {
                this.destinationRuleConfig[index].state = false
                if (this.destinationRuleConfig[index].trafficPolicy) {
                    if (this.destinationRuleConfig[index].trafficPolicy.connectionPool) {
                        this.destinationRuleConfig[index].trafficPolicy.connectionPool = ''
                        this.destinationRuleConfig[index].trafficPolicy.outlierDetection = ''
                    }
                }
                this.applyDestinationRule()
            },
            handleDestinationRuleDemoteSubsetOverrideLoadBalancerOpen(index) {
                this.destinationRuleDemoteSubsetLoadBalancerVisible = true
                var localLoadBalancer = 0
                if (this.destinationRuleConfig[index].trafficPolicy && this.destinationRuleConfig[index].trafficPolicy.loadBalancer) {
                    for (let i = 0; i < this.loadBalancerPolicy.length; ++i) {
                        if (this.loadBalancerPolicy[i] === this.destinationRuleConfig[index].trafficPolicy.loadBalancer.simple) {
                            localLoadBalancer = i
                        }
                    }
                }
                this.destinationRuleCurrentSubsetIndex = index
                this.destinationRuleDemoteSubsetLoadBalancerConfig = localLoadBalancer
            },
            handleDestinationRuleDemoteConfigOpen(index) {
                this.destinationRuleDemoteConfigVisible = true
                this.destinationRuleCurrentSubsetIndex = index
                var var1 = this.destinationRuleConfig[index]
                this.destinationRuleDemoteConfigForm.name = var1.name
                this.destinationRuleDemoteConfigForm.labels.version = var1.labels.version
                if (var1.trafficPolicy) {
                    if (var1.trafficPolicy.connectionPool) {
                        this.destinationRuleDemoteConfigForm.state = true
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.maxConnections = var1.trafficPolicy.connectionPool.tcp.maxConnections
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.connectTimeout = var1.trafficPolicy.connectionPool.tcp.connectTimeout
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.http1MaxPendingRequests = var1.trafficPolicy.connectionPool.http.http1MaxPendingRequests
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.maxRequestsPerConnection = var1.trafficPolicy.connectionPool.http.maxRequestsPerConnection
                    } else {
                        this.destinationRuleDemoteConfigForm.state = false
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.maxConnections = ''
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.connectTimeout = ''
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.http1MaxPendingRequests = ''
                        this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.maxRequestsPerConnection = ''
                    }
                    if (var1.trafficPolicy.loadBalancer) {
                        this.destinationRuleDemoteConfigForm.trafficPolicy.loadBalancer.simple = var1.trafficPolicy.loadBalancer.simple
                    } else {
                        this.destinationRuleDemoteConfigForm.trafficPolicy.loadBalancer.simple = ''
                    }
                } else {                        
                    this.destinationRuleDemoteConfigForm.state = false
                    this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.maxConnections = ''
                    this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.tcp.connectTimeout = ''
                    this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.http1MaxPendingRequests = ''
                    this.destinationRuleDemoteConfigForm.trafficPolicy.connectionPool.http.maxRequestsPerConnection = ''
                    this.destinationRuleDemoteConfigForm.trafficPolicy.loadBalancer.simple = ''
                }
            },
            handleEnvoyFilterDialogOpen(index) {
                this.envoyFilterDialogVisible = true
                this.envoyFilterStr = JSON.stringify(this.envoyFilters[index], null, '\t')
            },
            handleEnvoyFilterConfigOpen(index) {
                this.envoyFilterConfigVisible = true
                this.envoyFilterCurrentIndex = index
                var var1 = this.envoyFilters[index]
                this.envoyFilterConfigForm.metadata.name = var1.metadata.name
                this.envoyFilterConfigForm.spec.workloadSelector.labels.app = var1.spec.workloadSelector.labels.app
                if (var1.spec.workloadSelector.labels.version) {
                    this.envoyFilterConfigForm.spec.workloadSelector.labels.version = var1.spec.workloadSelector.labels.version
                } else {
                    this.envoyFilterConfigForm.spec.workloadSelector.labels.version = ''
                }
                this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens = var1.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens
                this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill = var1.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill
            },
            handleEnvoyFilterNewConfigOpen() {
                this.envoyFilterNewConfigVisible = true
                this.envoyFilterConfigForm.metadata.name = ''
                this.envoyFilterConfigForm.spec.workloadSelector.labels.app = ''
                this.envoyFilterConfigForm.spec.workloadSelector.labels.version = ''
                this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens = ''
                this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill = ''
            },
            getPods() {
                var _this = this
                var url = '/pods?clusterId=' + _this.clusterId + '&namespace=default&deployment='
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.pods = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("Pods列表查询出错")
                    }
                })
            },
            getDeployments() {
                var _this = this
                var url = '/deployments?clusterId=' + _this.clusterId + '&namespace=default'
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.deployments = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("Deployments列表查询出错")
                    }
                })
            },
            getStatefulSets() {
                var _this = this
                var url = '/statefulSets?clusterId=' + _this.clusterId + '&namespace=default'
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.statefulsets = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("StatefulSets列表查询出错")
                    }
                })
            },
            getServices() {
                var _this = this
                var url = '/services?clusterId=' + _this.clusterId
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.services = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("Services列表查询出错")
                    }
                })
            },
            getGateway() {
                var _this = this
                var url = '/gateway?clusterId=' + _this.clusterId
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.gateway[0] = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("Gateway信息查询出错")
                    }
                })
            },
            getVirtualService() {
                var _this = this
                var url = '/virtualService?clusterId=' + _this.clusterId
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.virtualService[0] = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("VirtualService信息查询出错")
                    }
                })
            },
            applyVirtualService() {
                var _this = this
                var versions = [ ]
                var weights = [ ]
                for (let i = 0; i < _this.virtualServiceConfig.length; ++i) {
                    const version = _this.virtualServiceConfig[i].destination.subset
                    const weight = _this.virtualServiceConfig[i].weight
                    versions[i] = version
                    weights[i] = weight
                }
                _this.$axios.post('virtualService/apply', {
                    clusterId: _this.clusterId,
                    namespace: _this.virtualService[0].metadata.namespace,
                    vsName: _this.virtualService[0].metadata.name,
                    versions: versions,
                    weights: weights
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('负载均衡配置成功！')
                        _this.getVirtualService()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('负载均衡配置出错！')
                    }
                })
                _this.handleVirtualServiceConfigOpen()
            },
            applyTemplateVirtualService() {
                var _this = this
                _this.$axios.post('virtualService/template', {
                    clusterId: _this.clusterId
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('Virtual Service模板配置成功！')
                        _this.getVirtualService()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('Virtual Service模板配置出错！')
                    }
                })
                _this.handleVirtualServiceConfigOpen()
            },
            getDestinationRule() {
                var _this = this
                var url = '/destinationRule?clusterId=' + _this.clusterId
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.destinationRule[0] = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("DestinationRule信息查询出错")
                    }
                })
            },
            applyDestinationRule() {
                var _this = this
                var policies = []
                for (let i = 0; i < _this.destinationRuleConfig.length; ++i) {
                    var policy = {
                        "version": '',
                        "state": '',
                        "tcp_maxConnections": '',
                        "tcp_connectTimeout": '',
                        "http_http1MaxPendingRequests": '',
                        "http_maxRequestPerConnection": '',
                        "consecutive5xxErrors": '',
                        "loadBalancer": ''
                    }
                    policy.version = _this.destinationRuleConfig[i].labels.version
                    policy.state = _this.destinationRuleConfig[i].state
                    if (_this.destinationRuleConfig[i].trafficPolicy) {
                        if (_this.destinationRuleConfig[i].trafficPolicy.loadBalancer) {
                            var currentPolicy = _this.destinationRuleConfig[i].trafficPolicy.loadBalancer.simple
                            policy.loadBalancer = 0
                            for (let i = 0; i < _this.loadBalancerPolicy.length; ++i) {
                                if (currentPolicy === _this.loadBalancerPolicy[i]) {
                                    policy.loadBalancer = i
                                }
                            }
                        }
                        if (_this.destinationRuleConfig[i].state) {
                            policy.tcp_maxConnections = _this.destinationRuleConfig[i].trafficPolicy.connectionPool.tcp.maxConnections
                            policy.tcp_connectTimeout = _this.destinationRuleConfig[i].trafficPolicy.connectionPool.tcp.connectTimeout.substring(0, _this.destinationRuleConfig[i].trafficPolicy.connectionPool.tcp.connectTimeout.length - 2)
                            policy.http_http1MaxPendingRequests = _this.destinationRuleConfig[i].trafficPolicy.connectionPool.http.http1MaxPendingRequests
                            policy.http_maxRequestPerConnection = _this.destinationRuleConfig[i].trafficPolicy.connectionPool.http.maxRequestsPerConnection
                            policy.consecutive5xxErrors = _this.destinationRuleConfig[i].trafficPolicy.outlierDetection.consecutive5xxErrors
                        }
                    }
                    policies[i] = policy
                }
                _this.$axios.post('destinationRule/apply', {
                    clusterId: _this.clusterId,
                    namespace: _this.destinationRule[0].metadata.namespace,
                    drName: _this.destinationRule[0].metadata.name,
                    globalLoadBalancer: _this.destinationRuleGlobalLoadBalancer,
                    policies: policies
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('配置成功！')
                        _this.getDestinationRule()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('配置出错！')
                    }
                })
                _this.handleDestinationRuleConfigOpen()
            },
            applyDestinationRuleGlobalLoadBalancer() {
                this.destinationRuleGlobalLoadBalancer = this.destinationRuleGlobalLoadBalancerConfig
                this.applyDestinationRule()
            },
            applyDestinationRuleOverrideLoadBalancer() {
                var index = this.destinationRuleCurrentSubsetIndex
                var trafficPolicy = {
                    "loadBalancer": {
                        "simple" : this.loadBalancerPolicy[this.destinationRuleDemoteSubsetLoadBalancerConfig]
                    }
                }
                this.destinationRuleConfig[index].trafficPolicy = trafficPolicy
                this.applyDestinationRule()
            },
            applyDestinationRuleDemote() {
                var index = this.destinationRuleCurrentSubsetIndex
                this.destinationRuleConfig[index] = this.destinationRuleDemoteConfigForm
                this.applyDestinationRule()
            },
            applyTemplateDestinationRule() {
                var _this = this
                _this.$axios.post('destinationRule/template', {
                    clusterId: _this.clusterId
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('Destination Rule模板配置成功！')
                        _this.getDestinationRule()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('Destination Rule模板配置出错！')
                    }
                })
                _this.handleDestinationRuleConfigOpen()
            },
            getEnvoyFilters() {
                var _this = this
                var url = '/envoyFilters?clusterId=' + _this.clusterId
                _this.$axios.get(url).then(successResponse => {
                    if (successResponse && successResponse.data.code === 200) {
                        _this.envoyFilters = successResponse.data.data
                    } else if (successResponse && successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error("限流配置列表查询出错")
                    }
                })
            },
            applyEnvoyFilter() {
                var _this = this
                var labels = {
                    "app": '',
                    "version": ''
                }
                labels.app = _this.envoyFilterConfigForm.spec.workloadSelector.labels.app
                labels.version = _this.envoyFilterConfigForm.spec.workloadSelector.labels.version
                _this.$axios.post('envoyFilter/apply', {
                    clusterId: _this.clusterId,
                    efName: _this.envoyFilterConfigForm.metadata.name,
                    state: true,
                    labels: labels,
                    max_tokens: _this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens,
                    tokens_per_fill: _this.envoyFilterConfigForm.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('配置成功！')
                        _this.getEnvoyFilters()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('配置出错！')
                    }
                })
            },
            deleteEnvoyFilter(index) {
                var _this = this
                _this.envoyFilterCurrentIndex = index
                var var1 = _this.envoyFilters[index]
                _this.$axios.post('envoyFilter/apply', {
                    clusterId: _this.clusterId,
                    efName: var1.metadata.name,
                    state: false,
                    labels: var1.spec.workloadSelector.labels,
                    max_tokens: var1.spec.configPatches[0].patch.value.typed_config.value.token_bucket.max_tokens,
                    tokens_per_fill: var1.spec.configPatches[0].patch.value.typed_config.value.token_bucket.tokens_per_fill
                }).then(successResponse => {
                    if (successResponse.data.code === 200) {
                        _this.$message.success('删除成功！')
                        _this.getEnvoyFilters()
                    } else if(successResponse.data.code === 400) {
                        _this.$message.warning(successResponse.data.message)
                    } else {
                        _this.$message.error('删除出错！')
                    }
                })
            }
        }
    }
</script>

<style scoped>

</style>
