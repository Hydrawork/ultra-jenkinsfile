//
// 需配置的环境变量
// 源码:
//   SCM_URL：代码下载地址
//   SCM_AUTH：版本管理登录信息(Jenkins凭证ID)
//   SCM_BRANCH：Git分支(仅Git用，SVN不管)
// 构建：
//   PRJ_NAME：项目整体名称
//   PRJ_MOD：项目模块名称
//   BUILD_MVN：自定义maven配置名称
//   BUILD_NPM：自定义npm配置名称
// 发布:
//   CAAS_REPO：=hub.ityy.ultrapower.com.cn:5050/test
//   CAAS_APP=YNIDC
//   CAAS_NS=ynidctest
//   CAAS_API=http://hub.ityy.ultrapower.com.cn:8090/caas
// 其他：
//   LINK_CONF：外部关联配置文件(properties格式)，用于批量引入环境变量。多个配置文件用半角分号隔开
pipeline {
    agent any
    parameters {
        booleanParam(name: 'STAGE_CLEAR', defaultValue: false, description: '清除工作空间重新构建')
        booleanParam(name: 'STAGE_PACKAGE', defaultValue: true, description: '构建完成后打包镜像')
        booleanParam(name: 'STAGE_TEST', defaultValue: false, description: '镜像打包后测试')
        booleanParam(name: 'STAGE_PUBLISH', defaultValue: true, description: '镜像打包后发布')
        booleanParam(name: 'STAGE_ARCHIVE', defaultValue: false, description: '镜像打包后归档')
    }
    stages {
        stage('环境准备') {
            steps {
                script {
                    if ( env.LINK_CONF != null) {
                        for (file in env.LINK_CONF.split(';')) {
                            try {
                                configFileProvider([
                                    configFile(fileId: file, variable: 'configfile')
                                ]) {
                                    echo "加载环境配置文件：${file}"
                                    readProperties(file: configfile).each {
                                        key, value ->
                                            if(env[key] == null) {
                                                env[key] = value
                                            }
                                    }
                                }
                            } catch( err ) {
                                error "配置文件 [ ${file} ] 不存在。";
                            }
                        }
                    }

                    echo '检查环境变量'
                    for ( key in ['SCM_URL', 'SCM_AUTH', 'PRJ_NAME', 'PRJ_MOD', 'CAAS_REPO', 'CAAS_APP', 'CAAS_NS', 'CAAS_API']) {
                        if (env[key] == null) {
                            error "缺少环境变量 [ ${key} ]";
                        }
                    }

                    if (env.PRJ_MOD == null) {
                        env.IMG_NAME = env.JOB_NAME
                    } else {
                        env.IMG_NAME = env.PRJ_MOD
                    }
                    echo "匹配发布名称：${env.IMG_NAME}"


                    echo "初始化maven构建环境"
                    if (env.BUILD_MVN == null) {
                        env.H2_HOME = "${env.JENKINS_HOME}/tools/hudson.tasks.Maven_MavenInstallation/maven"
                    } else {
                        env.H2_HOME = tool name: env.BUILD_MVN, type: 'maven'
                    }
                    echo "初始化nodejs构建环境"
                    if (env.BUILD_NPM == null) {
                        env.NODE_HOME = "${env.JENKINS_HOME}/tools/jenkins.plugins.nodejs.tools.NodeJSInstallation/nodejs"
                    } else {
                        env.NODE_HOME = tool name: env.BUILD_NPM, type: 'nodejs'
                    }
                    env.PATH = "${env.PATH}:${env.H2_HOME}/bin:${env.NODE_HOME}/bin"
                    sh label: '初始化Node', script: 
                    '''
                    if [ ! -f "${NODE_HOME}/bin/cnpm" ]; then
                        npm install cnpm -g --registry=https://registry.npm.taobao.org;
                    fi
                    '''

                    if (params.STAGE_CLEAR) {
                         sh label: '清除工作区', script: 'rm -fr ${WORKSPACE}/*'
                    }

                    env.SUB_TYPE = 'none'
                }

                sh label: '环境变量', script: 'set'
            }
        }
        stage('检出代码') {
            parallel {
                stage('主模块') {
                    steps {
                        checkout([
                            $class: 'SubversionSCM', 
                            additionalCredentials: [], 
                            excludedCommitMessages: '', 
                            excludedRegions: '', 
                            excludedRevprop: '', 
                            excludedUsers: '', 
                            filterChangelog: false, 
                            ignoreDirPropChanges: false, 
                            includedRegions: '', 
                            locations: [
                                [
                                    cancelProcessOnExternalsFail: true, 
                                    credentialsId: env.SCM_AUTH, 
                                    depthOption: 'infinity', 
                                    ignoreExternalsOption: true, 
                                    local: 'main', 
                                    remote: env.SCM_URL
                                ]
                            ], 
                            quietOperation: false, 
                            workspaceUpdater: [$class: 'UpdateWithCleanUpdater']
                        ])

                        script {
                            dir('main') {
                                env.MAIN_TYPE = sh returnStdout: true, script: 
                                '''
                                if [ -f pom.xml ]; then 
                                    echo -n maven;
                                elif [ -f package.json ]; then
                                    echo -n nodejs;
                                else
                                    echo -n none;
                                fi
                                '''
                            }
                        }
                        
                        echo '代码审计'
                    }
                }
                stage('子模块') {
                    when {
                        expression { env.SCM_SUB_URL != null }
                    }
                    steps {
                        checkout([
                            $class: 'SubversionSCM', 
                            additionalCredentials: [], 
                            excludedCommitMessages: '', 
                            excludedRegions: '', 
                            excludedRevprop: '', 
                            excludedUsers: '', 
                            filterChangelog: false, 
                            ignoreDirPropChanges: false, 
                            includedRegions: '', 
                            locations: [
                                [
                                    cancelProcessOnExternalsFail: true, 
                                    credentialsId: env.SCM_AUTH, 
                                    depthOption: 'infinity', 
                                    ignoreExternalsOption: true, 
                                    local: 'sub', 
                                    remote: env.SCM_SUB_URL
                                ]
                            ], 
                            quietOperation: false, 
                            workspaceUpdater: [$class: 'UpdateWithCleanUpdater']
                        ])

                        script {
                            dir('sub') {
                                env.SUB_TYPE = sh returnStdout: true, script: 
                                '''
                                if [ -f pom.xml ]; then 
                                    echo -n maven;
                                elif [ -f package.json ]; then
                                    echo -n nodejs;
                                else
                                    echo -n none;
                                fi
                                '''
                            }
                        }
                        
                        echo '代码审计'
                    }
                }
            }
        }
        stage('编译构建') {
            steps {
                script {
                    switch(env.SUB_TYPE) {
                        case 'maven':
                            dir('sub') {
                                sh label: 'maven编译', script: 'mvn clean install -DskipTests'
                            }
                            break;
                        case 'nodejs':
                            dir('sub') {
                                sh label: 'Node Install', script:
                                '''
                                if [ -d ../sub_node ]; then
                                    mv ../sub_node node_modules
                                fi
                                cnpm install
                                '''
                                try {
                                    sh label: 'Node Build', script: 'npm run build'
                                } finally {
                                    sh label: 'Node Install Cache', script: 'mv node_modules ../sub_node'
                                }
                            }
                            if (env.MAIN_TYPE == 'maven') {
                                env.SUB_PUB_DIR = 'src/main/resources/static/'
                                sh label: '复制结果', script: 'rm -fr main/${SUB_PUB_DIR}/* && cp -fr sub/dist/* main/${SUB_PUB_DIR}'
                            }
                        default:
                            break;
                    }
                    switch(env.MAIN_TYPE) {
                        case 'maven':
                            dir('main') {
                                sh label: 'maven编译', script: 'mvn clean package -DskipTests'
                            }
                            break;
                        case 'nodejs':
                            dir('main') {
                                sh label: 'Node Install', script:
                                '''
                                if [ -d ../main_node ]; then
                                    mv ../main_node node_modules;
                                fi
                                cnpm install
                                '''
                                try {
                                    sh label: 'Node Build', script: 'npm run build'
                                } finally {
                                    sh label: 'Node Install Cache', script: 'mv node_modules ../main_node'
                                }
                            }
                        default:
                            break;
                    }
                }
            }
        }
        stage('镜像打包') {
            when {
                expression { params.STAGE_PACKAGE }
            }
            steps {
                echo '准备镜像模板'

                echo "构建镜像：${IMG_NAME}"

                sh label: '转历史镜像', script: 
                '''
                if [ -n "$(docker images -q $PRJ_NAME/$IMG_NAME)" ]; then
                    docker tag $PRJ_NAME/$IMG_NAME history/$IMG_NAME
                    docker rmi $PRJ_NAME/$IMG_NAME
                fi
                '''

                script {
                    try {
                        dir('main') {
                            sh label: '构建镜像', script: 
                            '''
                            docker build -t $PRJ_NAME/$IMG_NAME --build-arg PACKAGE_FILE=target/$IMG_NAME.war .
                            '''
                        }
                        
                    } finally {
                        sh label: '清理历史镜像', script: 
                        '''
                        if [ -n "$(docker images -q history/$IMG_NAME)" ] && [ -z "$(docker ps -aqf ancestor=history/$IMG_NAME)" ]; then
                            docker rmi history/$IMG_NAME
                        fi
                        '''
                    }
                }
            }
        }
        stage('镜像测试') {
            when {
                expression { params.STAGE_TEST && params.STAGE_PACKAGE }
            }
            steps {
                echo '自动化测试'
            }
        }
        stage('镜像发布') {
            when {
                expression { params.STAGE_PACKAGE }
            }
            parallel {
                stage('部署上云') {
                    when {
                        expression { params.STAGE_PUBLISH }
                    }
                    steps {
                        script {
                            env.BUILD_VERSION = VersionNumber(
                                projectStartDate: '', 
                                versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY}', 
                                versionPrefix: '', 
                                worstResultForIncrement: 'SUCCESS'
                            )
                        }
                        
                        sh label: '发布镜像', script: 
                        '''
                        docker tag $PRJ_NAME/$IMG_NAME $CAAS_REPO/$IMG_NAME:$BUILD_VERSION
                        docker push $CAAS_REPO/$IMG_NAME:$BUILD_VERSION
                        docker rmi $CAAS_REPO/$IMG_NAME:$BUILD_VERSION
                        '''

                        script {
                            def imageInfo = sh returnStdout: true, script: '''
                                curl -s $CAAS_API/v2/api/registry/getPublicImages.do -X POST -d "appKey=$CAAS_APP&jsonData={\\"proName\\":\\"$CAAS_NS\\"}"
                            '''
                            def json = readJSON text : imageInfo
                            def versions = json.responseData[0].imageArray.find{
                                it.imageMark == "$IMG_NAME"
                            }.version.findResults{
                                it.tag
                            }
                            println versions.join('\n')
                        }

                        script {
                            def deployInfo = sh returnStdout: true, script: '''
                                curl -s $CAAS_API/v2/api/app/getDeploymentListByService.do -X POST -d "appKey=$CAAS_APP&namespace=$CAAS_NS&serviceName=$IMG_NAME"
                            '''
                            def json = readJSON text : deployInfo
                            env.DEPLOY_NAME = json[0].deploymentName

                            def updateInfo = sh returnStdout: true, script: '''
                                curl -s $CAAS_API/v2/api/app/updateImageEdition.do \
                                    -X POST -d "appKey=$CAAS_APP&serviceName=$IMG_NAME&namespace=$CAAS_NS&updateToimageEdition=$BUILD_VERSION&deploymentName=$DEPLOY_NAME"
                            '''
                            json = readJSON text : updateInfo
                            if(json.result != 'success') {
                                error json
                            }
                        }
                    }
                }
                stage('压缩归档') {
                    when {
                        expression { params.STAGE_ARCHIVE }
                    }
                    steps {
                        sh label: '镜像归档', script: 
                        '''
                        cd ${JENKINS_HOME}/packages
                        docker save -o $PRJ_NAME/$IMG_NAME.tar $PRJ_NAME/$IMG_NAME
                        gzip -f $PRJ_NAME/$IMG_NAME.tar
                        chmod 666 $PRJ_NAME/$IMG_NAME.tar.gz
                        '''
                    }
                }
            }
        }
        stage('垃圾清理') {
            steps {
                echo "清理构建垃圾"

                script {
                    if (params.STAGE_PACKAGE) {
                        sh label: '清理废镜像', script: 
                        '''
                        testinst=`docker ps -aqf ancestor=history/$IMG_NAME`
                        if [ -n "$testinst" ]; then
                            docker rm -f $testinst;
                        fi
                        if [ -n "$(docker images -q history/$IMG_NAME)" ]; then
                            docker rmi history/$IMG_NAME
                        fi
                        '''
                    }
                }
            }
        }
    }
}
