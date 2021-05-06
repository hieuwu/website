# Activity Lifecycle and ViewModel

Understanding Activity lifecycle and ViewModel helps you use the lifecycle callbacks more effective.
Below diagram is the running sequence of Activity lifecycle and ViewModel callback:


This is the implementation code of the diagram:

In Activity:

```c#
[Activity(Label = "MyActivity")]
    public class MyActivity : ReactiveActivity<MyViewModel>
    {
        protected override void OnCreate(Bundle savedInstanceState)
        {
            base.OnCreate(savedInstanceState);
            SetContentView(Resource.Layout.MyActivityLayout);
            ViewModel = new MyViewModel();
            this.WireUpControls();
            this.WhenActivated(Binding);
        }
        
        private void Binding(Action<IDisposable> d)
        {
            d(this.BindCommand(ViewModel, vm => vm.SomeCommand, v => v.SomeButton));
            d(this.WhenAnyObservable(v => v.ViewModel.SomeCommand)
                .Subscribe(_ => HandleButtonClick()));
        }
}
```

In ViewModel:
```c#
public class MyViewModel : ReactiveObject, ISupportsActivation
{
        public MyViewModel()
        {
            Activator = new ViewModelActivator();
            SomeCommand = ReactiveCommand.Create(SomeMethod);
            this.WhenActivated(Binding);
        }
        
        private void Binding(CompositeDisposable d)
        {
            this.WhenAnyValue(vm => vm.SomeObject).Subscribe(x =>
            {
               //Do something when your property change
               
            }).DisposeWith(d);            
            Disposable.Create(HandleDispose)
                .DisposeWith(d);
        }
        
        private void HandleDispose()
        {
          //Do whatever you want when your viewmodel dispose
        }
        
        private void SomeMethod() 
        {
        }
        
        private object SomeObject { get; set;};
        public ViewModelActivator Activator { get; }
       
}

```
When you first start an Activity, `OnCreate` is called.
Inside this callback, you construct your ViewModel, as the official guidline states.
Then inside the constructor method of ViewModel, you create commands and initiate value for propeties in ViewModel.
When the view is activated, your binding set up will be run, which is what you write in the `Binding(CompositeDisposable d)` in ViewModel.
Next, the `Binding(Action<IDisposable> d)` in the current Activity runs your set up. This methods includes binding set up for ViewModel to View or either betweens your view controller that this Activity contains
Then `OnResume` runs and your activity enters active state.
When navigation events occur like going to another Activity or hitting back button, the current Activity enters PAUSED state and the ViewModel is disposed, including its binding set up.

One thing you should notice is the case that your Activity contains RecylerView and Adapter.
Given that you have one list of data in the ViewModel let say it's `List<Data>`. And in the Activity, you set up a RecyclerView. And the list is displayed on screen.
If you navigate to another Activity, as the diagram shows, your ViewModel will be disposed. 
Then you back to the list Activity, because the ViewModel is already disposed, but the Adapter still there along with the old data displayed
(in fact there is no data). So when you interact with an item in the list, your app will crash with an exception


