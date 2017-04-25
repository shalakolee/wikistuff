# USEFUL STUFF


## WPF 

## using the INotifyPropertyChanged in models

this is good to bind the xaml to properties that will change without the need for code to update the UI.  I normally 
use this in the models and the viewmodels    

```
#!c#
using System.ComponentModel;
using System.Runtime.CompilerServices;
namespace customNamespace
{
    public class Class : INotifyPropertyChanged
    {
        private string _firstname;
        public string FirstName
        {
            get { return _firstname; }
            set
            {
                if (_firstname == value) return;
                _firstname= value;
                OnPropertyChanged();
            }
        }
   
        public event PropertyChangedEventHandler PropertyChanged;
        protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
        {
            PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
        }


    }
}

```
## using the INotifyPropertyChanged in viewmodels to watch changes in the entire object instead of each individual property

```
#!c#
//make our object
private OrderModel _order;
public OrderModel Order
{
     get { return _order; }
     set 
     {
          _order = value;
          OnPropertyChanged();
     }
}
//using the code like this, anytime the order object is set it will throw the event,    
// rather than having  the fields inside throw it... this is especially useful for poco classes...   

//for example, if i load an order i want the UI to update, but dont care about field updates to the   
//ui until i load a new order, this would be the best way to do it  
```





## Binding xaml to a viewmodel

```
#!xaml

<!-- add this to the window section with the rest -->
    xmlns:viewmodels="clr-namespace:yournamespace.ViewModels"

<!-- add this directly after the window declaration -->
    <Window.DataContext>
        <viewmodels:yourviewmodel />
    </Window.DataContext>

```


## Commands (Icommand)

### RelayCommand.cs
this seems to be the best relay command that i have found
create a relaycommand.cs and put the code here in it
```
#!c#
using System.Windows.Input;

namespace customnamespace
{
    public class RelayCommand : ICommand
    {

        readonly Action<object> _execute;
        readonly Predicate<object> _canExecute;

        public RelayCommand(Action<object> execute) : this(execute, null) { }
        public RelayCommand(Action<object> execute, Predicate<object> canExecute)
        {
            if (execute == null)
                throw new ArgumentNullException("execute");
            _execute = execute; _canExecute = canExecute;
        }
        
        public bool CanExecute(object parameter)
        {
            return _canExecute == null ? true : _canExecute(parameter);
        }
        public event EventHandler CanExecuteChanged
        {
            add { CommandManager.RequerySuggested += value; }
            remove { CommandManager.RequerySuggested -= value; }
        }
        public void Execute(object parameter) { _execute(parameter); }
    }
}
//codebehind usage:
        private RelayCommand _exitCommand;
        public RelayCommand ExitCommand
        {
            get
            {
                if (_exitCommand == null)
                    _exitCommand = new RelayCommand(e => ExecuteExitCommand(), e=> canExecuteBasicCommand(true));
                return _exitCommand;
            }
        }

        private void ExecuteExitCommand()
        {
            System.Windows.Application.Current.Shutdown();
        }

        public bool canExecuteBasicCommand(bool canExecute)
        {
            return canExecute;
        }

//code behind usage with parameter
        private RelayCommand _loadOrderCommand;
        public RelayCommand LoadOrderCommand
        {
            get
            {
                if (_loadOrderCommand == null)
                    _loadOrderCommand = new RelayCommand(param => LoadOrder(param), e => canExecuteBasicCommand(true));
                return _loadOrderCommand;
            }
        }
        public void LoadOrder(object orderNumber)
        {
            Order.OrderNumber = orderNumber.ToString();
        }


//xaml binding
<MenuItem Header="E_xit" Command="{Binding Path=ExitCommand}"  />

//xaml binding with parameter
<MenuItem x:Name="mnuLoad" Header="Load Specific Order" Command="{Binding Path=OrderViewModel.LoadOrderCommand}" CommandParameter="DM12345" />


```
