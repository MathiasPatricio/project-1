from operator import itemgetter # I use this module so it is easier to to extract elements from lists 
import datetime

def get_service_date(item):
    """Returns the service date for sorting."""
    return item[1]['service_date'] # Define the helper function before sorting and returns the service date for sorting
 
def read_file(filename):
    """Reads a file and returns a list of valid lines."""
    with open(filename, 'r') as file:
        return [line.strip().split(',') for line in file if line.strip()]  # Skip empty lines

def process_inventory(manufacturer_file, price_file, service_date_file):
    #Processes the input files and generates required inventory reports.
    # Read input files
    manufacturer_data = read_file(manufacturer_file)
    price_data = read_file(price_file)
    service_data = read_file(service_date_file)
    
    # Create dictionaries to store item details
    inventory = {}

    # Populate inventory with manufacturer data
    for item in manufacturer_data:
        if len(item) < 3:
            print(f"Skipping malformed line in {manufacturer_file}: {item}")  # Debugging info
            continue  # Skip bad data

        item_id, manufacturer, item_type = item[:3]
        damaged = item[3] if len(item) > 3 else ""
        inventory[item_id] = {
            'manufacturer': manufacturer,
            'item_type': item_type,
            'damaged': damaged
        }

    # Add price details
    for item in price_data:
        if len(item) < 2:
            continue
        item_id, price = item
        if item_id in inventory:
            inventory[item_id]['price'] = int(price)

    # Add service date details
    for item in service_data:
        if len(item) < 2:
            continue
        item_id, service_date = item
        if item_id in inventory:
            inventory[item_id]['service_date'] = datetime.datetime.strptime(service_date, "%m/%d/%Y")

    # Generate FullInventory.txt (Sorted by manufacturer name)
    with open("FullInventory.txt", 'w') as file:
        for item_id in sorted(inventory.keys(), key=itemgetter(0)):  # Sort by manufacturer name
            item = inventory[item_id]
            file.write(f"{item_id}, {item['manufacturer']}, {item['item_type']}, {item.get('price', 'N/A')}, "
                       f"{item.get('service_date', 'N/A').strftime('%m/%d/%Y') if 'service_date' in item else 'N/A'}, "
                       f"{item['damaged']}\n")

    # Generate Item type Inventory files (Sorted by item ID)
    item_types = {}
    for item_id, item in inventory.items():
        item_types.setdefault(item['item_type'], []).append((item_id, item))

    for item_type, items in item_types.items():
        with open(f"{item_type}Inventory.txt", 'w') as file:
            for item_id, item in sorted(items, key=itemgetter(0)):  # Sort by item ID
                file.write(f"{item_id}, {item['manufacturer']}, {item.get('price', 'N/A')}, "
                           f"{item.get('service_date', 'N/A').strftime('%m/%d/%Y') if 'service_date' in item else 'N/A'}, "
                           f"{item['damaged']}\n")

    # Generate PastServiceDateInventory.txt (Sorted by service date)
    today = datetime.datetime.today()
    past_service_items = [
        (item_id, item) for item_id, item in inventory.items()
        if 'service_date' in item and item['service_date'] < today
    ]

    with open("PastServiceDateInventory.txt", 'w') as file:
        for item_id, item in sorted(past_service_items, key=get_service_date):  # Replacing lambda with function
            file.write(f"{item_id}, {item['manufacturer']}, {item['item_type']}, {item.get('price', 'N/A')}, "
                   f"{item['service_date'].strftime('%m/%d/%Y')}, {item['damaged']}\n")

    # Generate DamagedInventory.txt (Sorted by price in descending order)
    damaged_items = [
        (item_id, item) for item_id, item in inventory.items() if item['damaged']
    ]

    with open("DamagedInventory.txt", 'w') as file:
        for item_id, item in sorted(damaged_items, key=itemgetter(1), reverse=True):  # Sort by price descending
            file.write(f"{item_id}, {item['manufacturer']}, {item['item_type']}, {item.get('price', 'N/A')}, "
                       f"{item.get('service_date', 'N/A').strftime('%m/%d/%Y') if 'service_date' in item else 'N/A'}\n")

if __name__ == "__main__":
    process_inventory("ManufacturerList.txt", "PriceList.txt", "ServiceDatesList.txt")
