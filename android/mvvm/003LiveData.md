# LiveData

[`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 是一种可观察的数据存储器类。与常规的可观察类不同，LiveData 具有生命周期感知能力，意指它遵循其他应用组件（如 Activity、Fragment 或 Service）的生命周期。这种感知能力可确保 LiveData 仅更新处于活跃生命周期状态的应用组件观察者。

如果观察者（由 [`Observer`](https://developer.android.com/reference/androidx/lifecycle/Observer) 类表示）的生命周期处于 [`STARTED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#STARTED) 或 [`RESUMED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#RESUMED) 状态，则 LiveData 会认为该观察者处于活跃状态。LiveData 只会将更新通知给活跃的观察者。为观察 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象而注册的非活跃观察者不会收到更改通知。

您可以注册与实现 [`LifecycleOwner`](https://developer.android.com/reference/androidx/lifecycle/LifecycleOwner) 接口的对象配对的观察者。有了这种关系，当相应的 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) 对象的状态变为 [`DESTROYED`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle.State#DESTROYED) 时，便可移除此观察者。这对于 Activity 和 Fragment 特别有用，因为它们可以放心地观察 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象，而不必担心泄露（当 Activity 和 Fragment 的生命周期被销毁时，系统会立即退订它们）。

### 使用 LiveData 的优势

使用 LiveData 具有以下优势：

- **确保界面符合数据状态**

  LiveData 遵循观察者模式。当底层数据发生变化时，LiveData 会通知 [`Observer`](https://developer.android.com/reference/androidx/lifecycle/Observer) 对象。您可以整合代码以在这些 `Observer` 对象中更新界面。这样一来，您无需在每次应用数据发生变化时更新界面，因为观察者会替您完成更新。

- **不会发生内存泄漏**

  观察者会绑定到 [`Lifecycle`](https://developer.android.com/reference/androidx/lifecycle/Lifecycle) 对象，并在其关联的生命周期遭到销毁后进行自我清理。

- **不会因 Activity 停止而导致崩溃**

  如果观察者的生命周期处于非活跃状态（如返回栈中的 Activity），则它不会接收任何 LiveData 事件。

- **不再需要手动处理生命周期**

  界面组件只是观察相关数据，不会停止或恢复观察。LiveData 将自动管理所有这些操作，因为它在观察时可以感知相关的生命周期状态变化。

- **数据始终保持最新状态**

  如果生命周期变为非活跃状态，它会在再次变为活跃状态时接收最新的数据。例如，曾经在后台的 Activity 会在返回前台后立即接收最新的数据。

- **适当的配置更改**

  如果由于配置更改（如设备旋转）而重新创建了 Activity 或 Fragment，它会立即接收最新的可用数据。

- **共享资源**

  您可以使用单例模式扩展 [`LiveData`](https://developer.android.com/reference/androidx/lifecycle/LiveData) 对象以封装系统服务，以便在应用中共享它们。`LiveData` 对象连接到系统服务一次，然后需要相应资源的任何观察者只需观察 `LiveData` 对象。如需了解详情，请参阅[扩展 LiveData](https://developer.android.com/topic/libraries/architecture/livedata#extend_livedata)。





https://developer.android.com/topic/libraries/architecture/livedata