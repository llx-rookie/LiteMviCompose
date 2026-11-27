# LiteMviCompose
🚀 一个极简、类型安全、完美契合 Jetpack Compose 的 MVI 架构纯源码库。


LiteMVI 的核心只有一个目的：用最少的代码（不到 200 行），提供最纯粹的单向数据流（UDF）体验。

零第三方依赖：只依赖官方 Compose 和 Lifecycle，没有 Retrofit、没有 Koin/Dagger、没有复杂的注解处理器。

完全源码公开：推荐直接下载源码放入你的项目中，你可以随意修改、删减。

极简 API：只需继承 DslViewModel，你的业务代码就能丝滑运转。

💻 快速上手 (Quick Start)
使用 LiteMVI 定义你的业务页面非常简单，只需三步：

1. 定义契约 (Contract)
定义 State（状态）、Intent（用户意图）、Effect（一次性事件）。

kotlin
// 1. 状态：必须是不可变的数据类
data class DemoState(val count: Int = 0) : UiState

// 2. 意图：用户的所有操作
sealed class DemoIntent : UiIntent {
    data object Increment : DemoIntent()
}

// 3. 副作用：只执行一次的事件（如弹 Toast、导航）
sealed class DemoEffect : UiEffect {
    data class ShowToast(val msg: String) : DemoEffect()
}

2. 编写 ViewModel
你的业务逻辑全部收拢在这里。

kotlin
class DemoViewModel : DslViewModel<DemoState, DemoIntent, DemoEffect>() {
    override fun initialState() = DemoState()

    override fun registerIntents() {
        intent<DemoIntent.Increment> {
            // 更新状态
            reduce { copy(count = count + 1) }
            // 发送一次性事件
            if (state.count >= 5) {
                effect { DemoEffect.ShowToast("已达到 5 次！") }
            }
        }
    }
}
3. 编写 UI
使用 MviContainer 将 ViewModel 和 Composable 绑定。

kotlin
@Composable
fun DemoScreen(viewModel: DemoViewModel) {
    MviContainer(
        viewModel = viewModel,
        onEffect = { effect ->
            when (effect) {
                is DemoEffect.ShowToast -> {
                    // 这里处理 Toast 或导航（由调用方决定）
                }
            }
        }
    ) { state, dispatch ->
        // UI 只依赖 state 进行渲染，dispatch 分发意图
        Column {
            Text("Count: ${state.count}")
            Button(onClick = { dispatch(DemoIntent.Increment) }) {
                Text("+1")
            }
        }
    }
}
