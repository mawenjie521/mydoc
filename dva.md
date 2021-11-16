### 说明

- 1.工程使用的是create-react-app 创建的基于typescript的react工程；

- 2.使用dva的状态管理模型；

- 3.使用redux-actions对action统一管理；

- 4.pc和移动端工程在同一个工程中，目录中的src/views/pc对应pc工程， src/views/mobile对应移动端工程；

- 5.pc和移动端分别打包，pc的html模版为public/index.html文件和入口为src/index.tsx文件；
    移动端的html模版为public/mobile.html文件和入口为src/mobile.tsx文件；

- 5.src/router为路由文件;

- 6.配置路径代理，先在config/paths.js中添加指定路径；再在config/webpack.config.js中指定alias；最后在tsconfig.json中配置；

### 开发说明

- 1.pc和mobile目录下再区分不同的应用，比如 translation应用

- 2.每个应用目录下都有一个pages和components目录，pages放页面，数据的处理逻辑，components放UI组件；

- 3.types目录统一放一些接口，全局接口放置在src/types里，每个应用可以建立自己的types；

- 4.models是state管理文件目录

- 5.actions是reduce调用的action的统一管理目录

- 6.router中有两个文件，router.js为pc端路由，mobileRouter.js为移动端路由；

### 为了区分环境变量，使用了handlebars模版来生产index.html

- 1.在执行包含执行changeConfig.js文件的命令时，根据传入的参数来区分正式还是测试环境；

- 2.再根据不同的环境使用./public/index.hbs模版加载pre.js或者prod.js生成./public/index.html文件；

- 3.index.html文件中包含了当前环境的一些全局变量；

- 4.修改pre.js或者prod.js文件之后，要重新执行包含执行changeConfig.js文件的命令；

### dva使用说明

- 1.创建dva生成组件，把组件挂载到dom上

    ```javascript
    // index.tsx 入口文件
    import dva from 'dva';
    import { createBrowserHistory as createHistory } from 'history';

    const app = dva({
        history: createHistory(),// 声明history之后，在子组件中可以直接调用history进行路由切换
    });

    app.model(require('./models').default); // 加载models对象

    app.router(require('./router').default);// 加载路由

    const App = app.start();// 生成组件

    ReactDOM.render(
        <React.StrictMode>
            <App />
        </React.StrictMode>,
        document.getElementById('root')
    );

- 2.创建models

    ```javascript
    // models/index.ts
    import { getInfo } from '../api';
    export default {
        namespace: 'app', // namespace是必须的，组件中只有models，就是靠这个namespace区分的
        state: {
            test: 'this is test',
            info: { result: 'sss' },
        },
        reducers: {
            changeTest(state: any, { payload }: { payload: any }) {
                return { ...state, test: payload };
            },
            setInfo(state: any, { payload }: any) {
                return { ...state, info: payload.content };
            },
        },
        effects: {
            *getInfo({ payload }: any, { call, put }: any): any {
                let result = null;
                try {
                    result = yield call(getInfo, payload);
                } catch (e: any) {
                    return;
                }
                yield put({
                    type: 'setInfo',
                    payload: result,
                });
                return true;
            },
        },
    };

- 3. 在组件中使用model

    ```javascript
    // home.tsx home组件
    import { connect } from 'dva';

    function HomePage(props: any) {
        return (
            <div>
                {props.value}
                {props.info.result}
            </div>
        );
    }

    function mapStateToProps({ app }: { app: any }) { // app 就是声明model时的namespace
        return {
            value: app.test,
            info: app.info,
        };
    }

    function mapDispatchToProps(dispatch: any) {
        return {
            changeTest(data: any) {
                return dispatch({ type: 'app/changeTest', payload: data }); // ‘app/changeTest’ 中的 app 是声明model时的namespace
            },
            getInfo(data: any) {
                return dispatch({ type: 'app/getInfo', payload: data });
            },
        };
    }

    export default connect(mapStateToProps, mapDispatchToProps)(HomePage);// 使用connect把state和dispatch放入组件的props中