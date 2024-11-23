import requests
import json
from datetime import datetime, timedelta
import math


def fetch_rrd_data(nagios_api_url, api_token, hosts, services):
    # Inisialisasi time berakhir
    #end_time = datetime(2024, 10, 28, 0, 0, 0)
    end_time = datetime.now().replace(hour=00, minute=00, second=00)

    start_time = end_time - timedelta(days=1)

    # Format start_time dan end_time sebagai UNIX timestamps
    start_timestamp = int(start_time.timestamp())
    end_timestamp = int(end_time.timestamp())

    # Membagi string services menjadi list individual services
    service_list = [service.strip() for service in services.split(',')]

    all_rrd_data = {}
    for host in hosts:
        host_rrd_data = {}
        for service in service_list:
            # Membangun URL API dengan parameter yang diperlukan untuk setiap service
            url = f"{nagios_api_url}"
            params = {
                "apikey": api_token,
                "host_name": host,
                "service_description": service,
                "start": start_timestamp,
                "end": end_timestamp
            }

            # Request HTTP GET
            response = requests.get(url, params=params)

            # Memeriksa apakah permintaan berhasil
            if response.status_code == 200:
                # Parsing data JSON
                data = response.json()

                # Ekstrak data dari response JSON
                meta = data.get('meta', {})

                # Bulatkan floating-point agar data V_value tidak berbentuk NaN
                if 'legend' in meta:
                    data_rows = data.get('data', {}).get('row', [])

                    # Inisialisasi list untuk T (timestamp) dan V (Values)
                    T_values = []
                    current_time = start_time
                    while current_time < end_time:
                        T_values.append(current_time.strftime('%Y-%m-%d %H:%M:%S'))
                        current_time += timedelta(minutes=1)  # Mengatur interval waktu pada T_values

                    V_values = [[] for _ in range(len(meta['legend']['entry']))]

                for entry in data_rows:
                    t = datetime.fromtimestamp(int(entry['t'])).strftime('%Y-%m-%d %H:%M:%S')

                    # Periksa jika service hanya memiliki satu nilai atau lebih
                    if isinstance(entry['v'], list):
                        for i, val in enumerate(entry['v']):
                            if i < len(V_values):
                                # Jika nilai adalah NaN, ganti dengan 0
                                V_values[i].append(float(val) if not math.isnan(float(val)) else 0)
                    else:
                        if V_values:
                            # Jika nilai adalah NaN, ganti dengan 0
                            V_values[0].append(float(entry['v']) if not math.isnan(float(entry['v'])) else 0)

                        # Append T to T_value


                    # Inisialisasi Value di setiap servicenya
                    value_1 = None
                    value_2 = None
                    value_3 = None
                    value_4 = None
                    value_5 = None
                    average_disk_1 = None
                    average_disk_2 = None
                    average_disk_3 = None
                    average_memory_1 = None
                    average_memory_2 = None
                    average_memory_3 = None
                    average_cpu = None

                    #cpu_value = None

                    # Penghitungan untuk mencari Average pada setiap value 
                    if service == "Mysql-database Disk Usage":
                        value_1 = V_values[0]
                        value_2 = V_values[1] 
                        value_3 = V_values[2]
                        average_disk_1 = round(sum(value_1) / len(value_1), 2) if value_1 else None
                        average_disk_2 = round(sum(value_2) / len(value_2), 2) if value_2 else None
                        average_disk_3 = round(sum(value_3) / len(value_3), 2) if value_3 else None

                    elif service == "Mysql-database Memory Usage":
                        value_1 = V_values[0]
                        value_2 = V_values[1] 
                        value_3 = V_values[2]
                        average_memory_1 = round(sum(value_1) / len(value_1), 2) if value_1 else None
                        average_memory_2 = round(sum(value_2) / len(value_2), 2) if value_2 else None
                        average_memory_3 = round(sum(value_3) / len(value_3), 2) if value_3 else None


                    elif service == "Mysql-database CPU Load":
                        value_1 = V_values[0]
                        average_cpu = round(sum(value_1) / len(value_1), 2) if value_1 else None

                    # Save data rrd
                    host_rrd_data[service] = {
                        "start_time": start_time.strftime('%Y-%m-%d %H:%M:%S'),
                        "end_time": end_time.strftime('%Y-%m-%d %H:%M:%S'),
                        "T_values": T_values,
                        "value_1": value_1,
                        "value_2": value_2,
                        "value_3": value_3,
                        "value_4": value_4,
                        "value_5": value_5,                    
                        "average_cpu" : average_cpu,
                        "average_disk_1" : average_disk_1,
                        "average_disk_2" : average_disk_2,
                        "average_disk_3" : average_disk_3,
                        "average_memory_1" : average_memory_1,
                        "average_memory_2" : average_memory_2,
                        "average_memory_3" : average_memory_3,
                        "meta": meta
                    }
            else:
                print(f"Failed to fetch RRD data for service {service} on host {host}. Status code: {response.status_code}")

        # Saving RRD data for this host
        all_rrd_data[host] = host_rrd_data
        

    return all_rrd_data

def send_to_google_sheets(all_rrd_data):
    script_url = "API Google Apps Script" #enpoint apps script

    payload = []

    for host, host_rrd_data in all_rrd_data.items():
        for service, rrd_data in host_rrd_data.items():
            start_time = rrd_data.get("start_time", "")
            end_time = rrd_data.get("end_time", "")
            cpu = rrd_data.get("average_cpu", "")
            usage_percentage = rrd_data.get("average_disk_1", "")
            usage_disk = rrd_data.get ("average_disk_2", "")
            limit_disk = rrd_data.get ("average_disk_3", "")
            usage_percentage_memory = rrd_data.get("average_memory_1", "")
            usage_memory = rrd_data.get ("average_memory_2", "")
            limit_memory = rrd_data.get ("average_memory_3")


            payload.append({
                "Host": host,
                "Start_Time": start_time,
                "End_Time": end_time,
                "CPU_Usage": cpu,
                "Percentage_Disk": usage_percentage,
                "Usage_Disk": usage_disk,
                "Limit_Disk": limit_disk,
                "Usage_Percentage_Memory" : usage_percentage_memory,
                "Usage_Memory" : usage_memory,
                "Limit_Memory" : limit_memory
            })

    single_payload = {
        "Host": payload[0]["Host"],  # Menggunakan host dari entri pertama
        "Start_Time": payload[0]["Start_Time"],  
        "End_Time": payload[0]["End_Time"],  
        "CPU_Usage": payload[0]["CPU_Usage"],
        "Percentage_Disk": payload[1]["Percentage_Disk"],
        "Usage_Disk" : payload[1]["Usage_Disk"],
        "Limit_Disk" : payload[1]["Limit_Disk"],
        "Usage_Percentage_Memory" : payload[2]["Usage_Percentage_Memory"],
        "Usage_Memory" : payload[2]["Usage_Memory"],
        "Limit_Memory" : payload[2]["Limit_Memory"]
    }

    print("Payload to be sent:")
    print(single_payload)  # Print payload for debugging purposes

    response = requests.post(script_url, json=single_payload)

    if response.status_code == 200:
        print("Data successfully sent to Google Sheets.") 
    else:
        print("Failed to send data to Google Sheets. Status code:", response.status_code)

if __name__ == "__main__":
    nagios_api_url = "http://x.x.x.x/nagiosxi/api/v1/objects/rrdexport"
    api_token = "API TOKEN NAGIOSXI"

    # List untuk hosts dan service harus sesuai sama yang dinagios nama host dan servicenya
    hosts = ["Mysql-database"]  #Nama Host
    services = "Mysql-database CPU Load, Mysql-database Disk Usage, Mysql-database Memory Usage"  #Nama Service 


    all_rrd_data = fetch_rrd_data(nagios_api_url, api_token, hosts, services)

    for host, host_rrd_data in all_rrd_data.items():
        print(f"RRD data for host: {host}")
        response_json = {
            "Host": host,
            "Start_Time": host_rrd_data["Mysql-Database CPU Load"]["start_time"], 
            "End_Time": host_rrd_data["Mysql-Database CPU Load"]["end_time"],
            "CPU_Stats": host_rrd_data["Mysql-Database CPU Load"]["average_cpu"],# isi sesuai sama services dan yang ditandain save rrd data
            "Usage_Percentage" : host_rrd_data["Mysql-Database Disk Usage"]["average_disk_1"], # isi sesuai sama services dan yang ditandain save rrd data
            "Usage_Disk" : host_rrd_data["Mysql-Database Disk Usage"]["average_disk_2"], # isi sesuai sama services dan yang ditandain save rrd data
            "Limit_Disk" : host_rrd_data["Mysql-Database Disk Usage"]["average_disk_3"], # isi sesuai sama services dan yang ditandain save rrd data
            "Usage_Percentage_Memory" : host_rrd_data["Mysql-Database Memory Usage"]["average_memory_1"], # isi sesuai sama services dan yang ditandain save rrd data
            "Usage_Memory" : host_rrd_data["Mysql-Database Memory Usage"]["average_memory_2"], # isi sesuai sama services dan yang ditandain save rrd data
            "Limit_Memory" : host_rrd_data["Mysql-Database Memory Usage"]["average_memory_3"] # isi sesuai sama services dan yang ditandain save rrd data

        }
        print(json.dumps(response_json, indent=4))

    send_to_google_sheets(all_rrd_data)
