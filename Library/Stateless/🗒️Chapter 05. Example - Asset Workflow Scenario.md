
# ✍️ Diagram
![[AssetWorkflow.png]]

# ✍️ How to use

### 🧩AssetWorkflow
---
```ad-note
collapse: close
title: AssetInformation.cs
~~~csharp
namespace AssetWorkflow
{
    public class AssetInformation
    {
        public AssetInformation(int assetID, string assetName = "")
        {
            AssetID = assetID;
            AssetName = assetName;
        }

        public int AssetID { get; set; }
        public string AssetName { get; set; }
        public Person Owner { get; set; }
    }
}
~~~
```

```ad-note
collapse: close
title: Person.cs
~~~csharp
namespace AssetWorkflow
{
    public class Person
    {
        public Person(int personID, string personName, string emailAddress)
        {
            PersonID = personID;
            PersonName = personName;
            EmailAddress = emailAddress;
        }

        public int PersonID { get; set; }
        public string PersonName { get; set; }
        public string EmailAddress { get; set; }
    }
}
~~~
```

```ad-note
collapse: open
title: Asset.cs
~~~csharp
using Stateless;
using System;

namespace AssetWorkflow;

public class Asset
{
    public enum State { New, Available, Allocated, UnderMaintenance, Unavailable, Decommissioned };
    public enum Trigger { Tested, Assigned, Released, RequestRepair, RequestUpdate, Transferred, Repaired, Lost, Discarded, Found };

    protected State _state;
    protected State _previousState;
    protected StateMachine<State, Trigger> _machine;

    protected StateMachine<State, Trigger>.TriggerWithParameters<Person> _assignTrigger;
    protected StateMachine<State, Trigger>.TriggerWithParameters<Person> _transferTrigger;

    public bool IsSuccesful { get; protected set; }
    public AssetInformation AssetData { get; protected set; }
    public State AssetState
    {
        get
        {
            return _state;
        }
        set
        {
            _previousState = _state;
            _state = value;
            Console.WriteLine("------------------------");
            Console.WriteLine($"Asset No : {AssetData.AssetID}");
            Console.WriteLine($"Previous asset state : {_previousState}");
            Console.WriteLine($"New asset state : {_state}");
        }
    }

    public Asset(AssetInformation data)
    {
        InitializeStateMachine();
        AssetData = data;
    }

    private void InitializeStateMachine()
    {
        _state = State.New;

        _machine = new StateMachine<State, Trigger>(() => AssetState, s => AssetState = s);

        _machine
            .Configure(State.New)
            .Permit(Trigger.Tested, State.Available)
            .OnEntry(() => OnEntry())
            .OnActivate(() => OnActivate())
            .Permit(Trigger.Lost, State.Unavailable)
            .OnDeactivate(() => OnDeactivate())
            .OnExit(() => OnExit());

        _machine
            .Configure(State.Available)
            .OnEntry(() => OnEntry())
            .OnActivate(() => OnActivate())
            .Permit(Trigger.Assigned, State.Allocated)
            .Permit(Trigger.Lost, State.Unavailable)
            .OnExit(() => OnExit())
            .OnEntryFrom(Trigger.Found, () => ProcessFound())
            .OnEntryFrom(Trigger.Released, () => ProcessDecommission())
            .OnDeactivate(() => OnDeactivate());

        _assignTrigger = _machine.SetTriggerParameters<Person>(Trigger.Assigned);
        _transferTrigger = _machine.SetTriggerParameters<Person>(Trigger.Transferred);

        _machine
            .Configure(State.Allocated)
            .OnEntry(() => OnEntry())
            .OnEntryFrom(_assignTrigger, owner => SetOwner(owner))
            .OnEntryFrom(_transferTrigger, owner => SetOwner(owner))
            .OnActivate(() => OnActivate())
            .OnExit(() => OnExit())
            .OnDeactivate(() => OnDeactivate())
            .PermitReentry(Trigger.Transferred)
            .Permit(Trigger.Released, State.Available)
            .Permit(Trigger.RequestRepair, State.UnderMaintenance)
            .Permit(Trigger.RequestUpdate, State.UnderMaintenance)
            .Permit(Trigger.Lost, State.Unavailable);

        _machine
            .Configure(State.UnderMaintenance)
            .OnEntry(() => OnEntry())
            .OnActivate(() => OnActivate())
            .OnExit(() => OnExit())
            .OnDeactivate(() => OnDeactivate())
            .Permit(Trigger.Repaired, State.Allocated)
            .Permit(Trigger.Lost, State.Unavailable)
            .Permit(Trigger.Discarded, State.Decommissioned);

        _machine
            .Configure(State.Unavailable)
            .OnEntry(() => OnEntry())
            .OnActivate(() => OnActivate())
            .OnExit(() => OnExit())
            .OnDeactivate(() => OnDeactivate())
            .PermitIf(Trigger.Found, State.Available, () => (_previousState != State.New))
            .PermitIf(Trigger.Found, State.New, () => (_previousState == State.New));

        _machine
            .Configure(State.Decommissioned)
            .OnEntry(() => ProcessDecommission())
            .OnActivate(() => OnActivate())
            .OnExit(() => OnExit())
            .OnDeactivate(() => OnDeactivate());
    } 

    #region Methods

    private void SetOwner(Person owner) { AssetData.Owner = owner; }

    private void ProcessDecommission()
    {
        Console.WriteLine("Clearing owner date..");
        AssetData.Owner = null;
        OnEntry();
    }

    private void ProcessFound()
    {
        if (AssetData.Owner != null)
        {
            Console.WriteLine("Clearing the owner data...");
            AssetData.Owner = null;
        }
    }

    public string GetDOTGraph() { return _machine.ToDotGraph(); }

    #endregion Methods

    #region Events

    private void OnEntry() { Console.WriteLine($"Entering {_state.ToString()} ..."); }
    private void OnActivate() { Console.WriteLine($"Activating {_state.ToString()} ..."); }
    private void OnDeactivate() { Console.WriteLine($"Deactivating {_state.ToString()} ..."); }
    private void OnExit() { Console.WriteLine($"Exiting {_state.ToString()} ..."); }

    #endregion Events

    #region Trigger wrapper

    private void Fire(Trigger trigger)
    {
        IsSuccesful = false;

        try
        {
            _machine.Fire(trigger);
            IsSuccesful = true;
        }
        catch
        {
            Console.WriteLine("Error during state transition.");
            IsSuccesful = false;
        }
    }

    public void Assign(Person owner)
    {
        IsSuccesful = false;

        try
        {
            _machine.Fire(_assignTrigger, owner);
            IsSuccesful = true;
        }
        catch
        {
            Console.WriteLine("Error during state transition.");
            IsSuccesful = false;
        }
    }

    public void Transfer(Person owner)
    {
        IsSuccesful = false;

        try
        {
            _machine.Fire(_transferTrigger, owner);
            IsSuccesful = true;
        }
        catch
        {
            Console.WriteLine("Error during state transition.");
            IsSuccesful = false;
        }
    }

    public void Discard() { Fire(Trigger.Discarded); }
    public void FinishedTesting() { Fire(Trigger.Tested); }
    public void Found() { Fire(Trigger.Found); }
    public void Lost() { Fire(Trigger.Lost); }
    public void Release() { Fire(Trigger.Released); }
    public void Repaired() { Fire(Trigger.Repaired); }
    public void RequestRepair() { Fire(Trigger.RequestRepair); }
    public void RequestUpdate() { Fire(Trigger.RequestUpdate); }

    #endregion Trigger wrapper
}
~~~
```

```ad-note
collapse: open
title: Program.cs
~~~csharp
using System;
using System.Collections.Generic;
using System.Text;

namespace AssetWorkflow;

class Program
{
    private static List<Asset> assetList = new List<Asset>();
    public static int counter = 1;

    static void Main()
    {
        int choice = 0;
        while (choice != 4)
        {
            choice = __showMainMenu();
            switch (choice)
            {
                case 1: Console.WriteLine("Creating asset.."); CreateAsset(); break;
                case 2: Console.WriteLine("Managing assets..."); ManageAssets(); break;
                case 3: Console.WriteLine("Listing assets..."); ListAssets(); break;
                case 4: Console.WriteLine("Exiting....."); break;
                default: break;
            }
        }

        #region Main - Local Function

        int __showMainMenu()
        {
            int choice;
            Console.WriteLine("Asset Manager v1");

            do
            {
                Console.WriteLine("Menu : (1) Create Asset  (2) Manage Asset (3) List Assets (4)Exit");

                string ch = Console.ReadLine();
                int.TryParse(ch, out choice);

            } while (choice < 1 || choice > 4);

            return choice;
        }

        #endregion
    }

    private static void CreateAsset()
    {
        AssetInformation assInfo = new AssetInformation(counter);
        counter++;
        assInfo.AssetName = "Asset " + assInfo.AssetID.ToString();

        Asset tempAsset = new Asset(assInfo);
        assetList.Add(tempAsset);
    }

    private static void ManageAssets()
    {
        string assetID;
        ListAssets();

        Console.WriteLine("Select asset to manage.");
        Console.Write("Asset ID :");
        assetID = Console.ReadLine();

        string valid = "ABCDEFGHIJK";

        if (__isAssetExist(assetID))
        {
            string choice = "X";
            Console.WriteLine("Asset Management");

            do
            {
                Console.WriteLine("Menu : (A) Test (B) Assign (C) Repair (D) Upgrade (E) Release (F) Transfer (G) Repaired (H) Discard (I) Lost (J) Found (K) Exit");

                choice = Console.ReadLine().ToUpper();

            } while (!valid.Contains(choice));

            Asset asset = assetList.Find(i => i.AssetData.AssetID.ToString() == assetID);

            if (choice == "K")
            {
                Console.WriteLine("Exiting asset management.");
                Console.WriteLine(asset.GetDOTGraph());
            }
            else
            {
                __manageAsset(asset, choice);
            }
        }
        else
        {
            Console.WriteLine("Asset not found.");
        }

        #region ManageAssets - Local Function

        bool __isAssetExist(string assetID)
        {
            int id = 0;
            int.TryParse(assetID, out id);

            return assetList.Exists(i => i.AssetData.AssetID == id);
        }

        void __manageAsset(Asset asset, string action)
        {
            switch (action)
            {
                case "A": asset.FinishedTesting(); break;
                case "B": asset.Assign(___GetOwner()); break;
                case "C": asset.RequestRepair(); break;
                case "D": asset.RequestUpdate(); break;
                case "E": asset.Release(); break;
                case "F": asset.Transfer(___GetOwner()); break;
                case "G": asset.Repaired(); break;
                case "H": asset.Discard(); break;
                case "I": asset.Lost(); break;
                case "J": asset.Found(); break;
                default: break;
            }

            Console.WriteLine($"Success : {asset.IsSuccesful}");

            Person ___GetOwner()
            {
                int id = counter;
                Console.WriteLine("Enter name : ");

                string name = Console.ReadLine();
                string email = name.Replace(" ", "") + "@email.com";
                counter++;

                return new Person(id, name, email);
            }
        }

        #endregion
    }

    private static void ListAssets()
    {
        foreach (Asset asset in assetList)
        {
            StringBuilder data = new StringBuilder();
            data.Append(asset.AssetData.AssetID.ToString() + " : ");
            data.Append(asset.AssetData.AssetName + " : ");

            if (asset.AssetData.Owner != null)
            {
                data.Append(asset.AssetData.Owner.EmailAddress + " : ");
            }

            data.Append(asset.AssetState.ToString());

            Console.WriteLine(data.ToString());
        }
    }
}
~~~
```
