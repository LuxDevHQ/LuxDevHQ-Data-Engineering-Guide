```python
from confluent_kafka import Producer
from faker import Faker
import random
import time
import datetime
import json

# Kafka Configuration
KAFKA_BROKER = "localhost:9092"  # Change to match your Kafka setup
TOPIC = "kenyan_users"

# Create a Kafka Producer instance
producer = Producer({'bootstrap.servers': KAFKA_BROKER})

# Create a Faker instance with British English locale
fake = Faker('en_GB')

def generate_kenyan_phone():
    prefixes = ['0700', '0701', '0702', '0703', '0704', '0705', '0706', '0707', '0708', '0709',
                '0710', '0711', '0712', '0713', '0714', '0715', '0716', '0717', '0718', '0719',
                '0720', '0721', '0722', '0723', '0724', '0725', '0726', '0727', '0728', '0729',
                '0730', '0731', '0732', '0733', '0734', '0735', '0736', '0737', '0738', '0739',
                '0740', '0741', '0742', '0743', '0744', '0745', '0746', '0747', '0748', '0749',
                '0750', '0751', '0752', '0753', '0754', '0755', '0756', '0757', '0758', '0759',
                '0760', '0761', '0762', '0763', '0764', '0765', '0766', '0767', '0768', '0769',
                '0770', '0771', '0772', '0773', '0774', '0775', '0776', '0777', '0778', '0779',
                '0790', '0791', '0792', '0793', '0794', '0795', '0796', '0797', '0798', '0799']
    
    prefix = random.choice(prefixes)
    suffix = ''.join(random.choices('0123456789', k=6))
    return f"{prefix} {suffix}"

def generate_kenyan_amount():
    amount = random.randint(100, 100000)
    return f"KES {amount:,}"

def generate_user():
    name = fake.name()
    domain = random.choice(['gmail.com', 'yahoo.com', 'hotmail.com', 'outlook.com', 'ke-mail.com'])
    email = f"{name.lower().replace(' ', '.').replace('.', random.choice(['.','-','_']))}{random.randint(1, 99)}@{domain}"
    phone = generate_kenyan_phone()
    amount = generate_kenyan_amount()
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    
    return {
        "timestamp": timestamp,
        "name": name,
        "email": email,
        "phone": phone,
        "amount": amount
    }

# Callback function to confirm message delivery
def delivery_report(err, msg):
    if err is not None:
        print(f"❌ Message delivery failed: {err}")
    else:
        print(f"✅ Message delivered to {msg.topic()} [{msg.partition()}]")

def main():
    print("Kafka Producer: Sending Kenyan user data to topic 'kenyan_users' every 5 seconds...")
    
    try:
        while True:
            user = generate_user()
            user_data = json.dumps(user)  # Convert to JSON format
            
            # Send data to Kafka
            producer.produce(TOPIC, value=user_data, callback=delivery_report)
            producer.flush()  # Ensure message is sent
            
            print(f"📤 Sent: {user_data}")

            time.sleep(5)
    
    except KeyboardInterrupt:
        print("\n🚪 Stopping Kafka producer.")
    
    finally:
        producer.flush()

if __name__ == "__main__":
    main()
```
