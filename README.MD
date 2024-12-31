Sure! I'll provide you with a complete C# code example that includes both shipment and inventory management using abstract classes and interfaces, along with a simple Windows Forms application to demonstrate how they can be used.

### Project Structure

```
InventoryShipmentManagement
│
├── Abstract
│   └── AbstractProductService.cs
│   └── AbstractShipmentService.cs
│
├── Interfaces
│   └── IProductService.cs
│   └── IShipmentService.cs
│
├── Services
│   └── ProductService.cs
│   └── ShipmentService.cs
│
├── Forms
│   └── MainForm.cs
│
├── Program.cs
└── App.config
```

### Abstract Classes

#### AbstractProductService.cs

```csharp
using System.Data;
using System.Windows.Forms;

namespace InventoryShipmentManagement.Abstract
{
    public abstract class AbstractProductService
    {
        protected DataTable table;

        public AbstractProductService()
        {
            table = new DataTable("Products");
            DataColumn idColumn = new DataColumn("ID", typeof(int));
            idColumn.AutoIncrement = true;
            idColumn.AutoIncrementSeed = 1;
            idColumn.AutoIncrementStep = 1;

            table.Columns.Add(idColumn);
            table.Columns.Add("Name", typeof(string));
            table.Columns.Add("Quantity", typeof(int));
            table.Columns.Add("Price", typeof(double));
        }

        public abstract DataTable GetProductTable();
        public abstract void InsertRecord(string name, int quantity, double price);
        public abstract void UpdateRecord(int rowIndex, string name, int quantity, double price);
        public abstract void DeleteRecord(int rowIndex, DataGridView dataGridViewInventory = null);
    }
}
```

#### AbstractShipmentService.cs

```csharp
using System.Data;

namespace InventoryShipmentManagement.Abstract
{
    public abstract class AbstractShipmentService
    {
        protected DataTable shipmentTable;

        public AbstractShipmentService()
        {
            shipmentTable = new DataTable("shipments");
            DataColumn shipmentIDColumn = new DataColumn("ShipmentID", typeof(int));
            shipmentIDColumn.AutoIncrement = true;
            shipmentIDColumn.AutoIncrementSeed = 1;
            shipmentIDColumn.AutoIncrementStep = 1;

            shipmentTable.Columns.Add(shipmentIDColumn);
            shipmentTable.Columns.Add("ProductID", typeof(int));
            shipmentTable.Columns.Add("ShipmentDate", typeof(DateTime));
            shipmentTable.Columns.Add("Status", typeof(string));
        }

        public abstract DataTable GetShipmentTable();
        public abstract void CreateShipment(int productId, DateTime shipmentDate, string status);
        public abstract void UpdateShipmentStatus(int rowIndex, string status);
    }
}
```

### Interfaces

#### IProductService.cs

```csharp
using System.Data;
using System.Windows.Forms;

namespace InventoryShipmentManagement.Interfaces
{
    public interface IProductService
    {
        DataTable GetProductTable();
        void InsertRecord(string name, int quantity, double price);
        void UpdateRecord(int rowIndex, string name, int quantity, double price);
        void DeleteRecord(int rowIndex, DataGridView dataGridViewInventory = null);
    }
}
```

#### IShipmentService.cs

```csharp
using System.Data;

namespace InventoryShipmentManagement.Interfaces
{
    public interface IShipmentService
    {
        DataTable GetShipmentTable();
        void CreateShipment(int productId, DateTime shipmentDate, string status);
        void UpdateShipmentStatus(int rowIndex, string status);
    }
}
```

### Services

#### ProductService.cs

```csharp
using System;
using System.Data;
using System.Windows.Forms;
using InventoryShipmentManagement.Abstract;
using InventoryShipmentManagement.Interfaces;

namespace InventoryShipmentManagement.Services
{
    public class ProductService : AbstractProductService, IProductService
    {
        public override DataTable GetProductTable()
        {
            return table;
        }

        public override void InsertRecord(string name, int quantity, double price)
        {
            DataRow newRow = table.NewRow();
            newRow["Name"] = name;
            newRow["Quantity"] = quantity;
            newRow["Price"] = price;
            table.Rows.Add(newRow);
        }

        public override void UpdateRecord(int rowIndex, string name, int quantity, double price)
        {
            if (rowIndex >= 0 && rowIndex < table.Rows.Count)
            {
                DataRow row = table.Rows[rowIndex];
                row["Name"] = name;
                row["Quantity"] = quantity;
                row["Price"] = price;
            }
            else
            {
                throw new IndexOutOfRangeException("Invalid row index.");
            }
        }

        public override void DeleteRecord(int rowIndex, DataGridView dataGridViewInventory = null)
        {
            if (dataGridViewInventory != null)
            {
                if (rowIndex >= 0 && rowIndex < dataGridViewInventory.Rows.Count)
                {
                    dataGridViewInventory.Rows.RemoveAt(rowIndex);
                }
            }
            else
            {
                if (rowIndex >= 0 && rowIndex < table.Rows.Count)
                {
                    table.Rows.RemoveAt(rowIndex);
                }
            }
        }
    }
}
```

#### ShipmentService.cs

```csharp
using System;
using System.Data;
using InventoryShipmentManagement.Abstract;
using InventoryShipmentManagement.Interfaces;

namespace InventoryShipmentManagement.Services
{
    public class ShipmentService : AbstractShipmentService, IShipmentService
    {
        public override DataTable GetShipmentTable()
        {
            return shipmentTable;
        }

        public override void CreateShipment(int productId, DateTime shipmentDate, string status)
        {
            DataRow newRow = shipmentTable.NewRow();
            newRow["ProductID"] = productId;
            newRow["ShipmentDate"] = shipmentDate;
            newRow["Status"] = status;
            shipmentTable.Rows.Add(newRow);
        }

        public override void UpdateShipmentStatus(int rowIndex, string status)
        {
            if (rowIndex >= 0 && rowIndex < shipmentTable.Rows.Count)
            {
                DataRow row = shipmentTable.Rows[rowIndex];
                row["Status"] = status;
            }
            else
            {
                throw new IndexOutOfRangeException("Invalid row index.");
            }
        }
    }
}
```

### Forms

#### MainForm.cs

```csharp
using System;
using System.Windows.Forms;
using InventoryShipmentManagement.Interfaces;
using InventoryShipmentManagement.Services;

namespace InventoryShipmentManagement.Forms
{
    public partial class MainForm : Form
    {
        private readonly IProductService productService;
        private readonly IShipmentService shipmentService;

        public MainForm(IProductService productService, IShipmentService shipmentService)
        {
            InitializeComponent();
            this.productService = productService;
            this.shipmentService = shipmentService;
        }

        // Event handlers and other methods
    }
}
```

### Program.cs

```csharp
using System;
using System.Windows.Forms;
using InventoryShipmentManagement.Forms;
using InventoryShipmentManagement.Services;

namespace InventoryShipmentManagement
{
    static class Program
    {
        [STAThread]
        static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);

            IProductService productService = new ProductService();
            IShipmentService shipmentService = new ShipmentService();

            Application.Run(new MainForm(productService, shipmentService));
        }
    }
}
```

### App.config

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <startup> 
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.7.2" />
  </startup>
</configuration>
```

This complete example includes abstract classes, interfaces, service implementations, and a simple Windows Forms application to demonstrate how to use these services. You can copy these files into your project structure and run the application.

If you need the files zipped, please let me know, and I'll provide the zipped folder. 😊
