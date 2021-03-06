## Example: a simple linux virtual machine

- This example dependes on the: [Example - basic network setup](Terraform_example_0.md)


*Note:* This examples requires several properties that needs to be set, see more information in the [Github repo](https://github.com/aberner/iac-azure-terraform-example)

```
#Get a Static Public IP
resource "azurerm_public_ip" "azure-web-ip" {
  name                = "${var.app_name}-${var.app_environment}-web-ip"
  location            = azurerm_resource_group.azure-rg.location
  resource_group_name = azurerm_resource_group.azure-rg.name
  allocation_method   = "Static"

  tags = {
    environment = var.app_environment,
    responsible = var.department_id
  }
}

#Create Network Card for Web Server VM
resource "azurerm_network_interface" "azure-web-nic" {
  name                = "${var.app_name}-${var.app_environment}-web-nic"
  location            = azurerm_resource_group.azure-rg.location
  resource_group_name = azurerm_resource_group.azure-rg.name

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.azure-subnet.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.azure-web-ip.id
  }

  tags = {
    environment = var.app_environment,
    responsible = var.department_id
  }
}

# Create web server vm
resource "azurerm_linux_virtual_machine" "azure-web-vm" {
  name                             = "${var.app_name}-${var.app_environment}-web-vm"
  location                         = azurerm_resource_group.azure-rg.location
  resource_group_name              = azurerm_resource_group.azure-rg.name
  network_interface_ids            = [azurerm_network_interface.azure-web-nic.id]
  size                             = "Standard_B1s"

  computer_name  = var.linux_vm_hostname
  admin_username = var.linux_admin_user
  admin_password = var.linux_admin_password
  disable_password_authentication = false

  source_image_reference {
    publisher = var.ubuntu-linux-publisher
    offer     = var.ubuntu-linux-offer
    sku       = var.ubuntu-linux-18-sku
    version   = "latest"
  }

  os_disk {
    name              = "${var.app_name}-${var.app_environment}-web-vm-os-disk"
    caching           = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  tags = {
    environment = var.app_environment,
    responsible = var.department_id
  }
}

#Output
output "external-ip-azure-web-server" {
  value = azurerm_public_ip.azure-web-ip.ip_address
}
```