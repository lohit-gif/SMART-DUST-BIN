# SMART-DUST-BIN
A Smart Dustbin is an automated waste management system that opens its lid using sensors, reducing human contact and improving hygiene. It uses a microcontroller like Arduino Uno to operate. Advanced versions can detect waste levels, support segregation, and enable efficient, eco-friendly disposal.
"""
Smart Dustbin System - IoT Enabled Waste Management
Features: Automatic lid opening, fill level monitoring, odor detection, remote alerts
"""

import time
import random
import threading
from datetime import datetime
from typing import Dict, List, Tuple

class SensorHub:
    """Simulates sensor readings for smart dustbin"""
    
    def __init__(self):
        self.ultrasonic_distance = 50  # cm
        self.fill_level = 0  # percentage
        self.odor_level = 0  # ppm
        self.temperature = 25  # celsius
        self.weight = 0  # kg
        
    def read_ultrasonic(self) -> float:
        """Simulate ultrasonic sensor for distance measurement"""
        # Person approaching simulation
        self.ultrasonic_distance = random.uniform(10, 100)
        return self.ultrasonic_distance
    
    def read_fill_level(self) -> int:
        """Simulate fill level sensor"""
        # Gradual fill over time
        if self.fill_level < 100:
            self.fill_level += random.uniform(0, 0.5)
        return int(min(100, self.fill_level))
    
    def read_odor(self) -> int:
        """Simulate gas sensor for odor detection"""
        # Odor increases with fill level
        base_odor = (self.fill_level / 100) * 500
        self.odor_level = base_odor + random.uniform(-20, 50)
        return int(max(0, min(1000, self.odor_level)))
    
    def read_temperature(self) -> float:
        """Simulate temperature sensor"""
        self.temperature += random.uniform(-0.5, 0.5)
        return round(self.temperature, 1)

class SmartDustbin:
    """Main smart dustbin controller"""
    
    def __init__(self, bin_id: str, capacity: float = 50):
        self.bin_id = bin_id
        self.capacity = capacity  # kg
        self.is_lid_open = False
        self.sensors = SensorHub()
        self.alert_thresholds = {
            'fill': 80,  # percentage
            'odor': 600,  # ppm
            'temperature': 45  # celsius
        }
        self.status_history: List[Dict] = []
        self.is_running = True
        
    def open_lid(self) -> None:
        """Open dustbin lid"""
        if not self.is_lid_open:
            self.is_lid_open = True
            print(f"[{datetime.now().strftime('%H:%M:%S')}] Lid OPENED")
            
    def close_lid(self) -> None:
        """Close dustbin lid"""
        if self.is_lid_open:
            self.is_lid_open = False
            print(f"[{datetime.now().strftime('%H:%M:%S')}] Lid CLOSED")
    
    def check_obstacle(self) -> bool:
        """Check if obstacle is detected near bin"""
        distance = self.sensors.read_ultrasonic()
        if distance < 30:  # Person detected within 30cm
            self.open_lid()
            return True
        else:
            if self.is_lid_open:
                # Close after 3 seconds if no one nearby
                time.sleep(3)
                if self.sensors.read_ultrasonic() >= 30:
                    self.close_lid()
            return False
    
    def monitor_fill_level(self) -> bool:
        """Monitor bin fill level and trigger alerts"""
        fill = self.sensors.read_fill_level()
        if fill >= self.alert_thresholds['fill']:
            print(f"⚠️  ALERT: Bin {self.bin_id} is {fill}% full! Need emptying")
            return True
        return False
    
    def monitor_odor(self) -> bool:
        """Monitor odor levels"""
        odor = self.sensors.read_odor()
        if odor >= self.alert_thresholds['odor']:
            print(f"⚠️  ALERT: High odor level detected ({odor} ppm) in bin {self.bin_id}")
            return True
        return False
    
    def get_status(self) -> Dict:
        """Get current bin status"""
        status = {
            'timestamp': datetime.now(),
            'bin_id': self.bin_id,
            'fill_level': self.sensors.read_fill_level(),
            'odor_level': self.sensors.read_odor(),
            'temperature': self.sensors.read_temperature(),
            'lid_status': 'open' if self.is_lid_open else 'closed'
        }
        self.status_history.append(status)
        return status
    
    def display_status(self) -> None:
        """Display current status"""
        status = self.get_status()
        print("\n" + "="*50)
        print(f"📊 SMART BIN STATUS - {status['timestamp'].strftime('%Y-%m-%d %H:%M:%S')}")
        print(f"Bin ID: {status['bin_id']}")
        print(f"Fill Level: {status['fill_level']}%")
        print(f"Odor Level: {status['odor_level']} ppm")
        print(f"Temperature: {status['temperature']}°C")
        print(f"Lid: {status['lid_status']}")
        print(f"Capacity Used: {(status['fill_level']/100)*self.capacity:.1f}/{self.capacity} kg")
        print("="*50)
    
    def auto_clean(self) -> None:
        """Simulate automatic cleaning cycle"""
        print("\n🧹 Starting auto-cleaning cycle...")
        time.sleep(2)
        print("✅ Cleaning completed!")
        self.sensors.odor_level = 0
        print("Odor neutralized successfully!")
    
    def run(self) -> None:
        """Main control loop"""
        print(f"🚀 Smart Dustbin {self.bin_id} initialized")
        print(f"Capacity: {self.capacity} kg")
        print(f"Alert thresholds: Fill {self.alert_thresholds['fill']}%, Odor {self.alert_thresholds['odor']} ppm")
        print("Press Ctrl+C to stop\n")
        
        try:
            while self.is_running:
                # Check for approaching person
                self.check_obstacle()
                
                # Monitor bin status
                fill_alert = self.monitor_fill_level()
                odor_alert = self.monitor_odor()
                
                # Display status every 10 cycles
                if int(time.time()) % 10 == 0:
                    self.display_status()
                
                # Auto-clean if odor threshold exceeded
                if self.sensors.read_odor() >= self.alert_thresholds['odor']:
                    self.auto_clean()
                
                # Send alert for full bin
                if fill_alert:
                    self.send_alert("Bin full! Immediate emptying required!")
                
                time.sleep(2)  # Check every 2 seconds
                
        except KeyboardInterrupt:
            print("\n\n🛑 Shutting down smart dustbin system...")
            self.cleanup()
    
    def send_alert(self, message: str) -> None:
        """Simulate sending alert to maintenance team"""
        print(f"\n📱 ALERT to maintenance: {message}")
        print(f"📍 Location: Bin {self.bin_id}")
        print(f"⏰ Time: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    
    def cleanup(self) -> None:
        """Cleanup before shutdown"""
        self.is_running = False
        if self.is_lid_open:
            self.close_lid()
        print(f"📈 Total status records: {len(self.status_history)}")
        print("✅ System shutdown complete. Goodbye!")

class MonitoringServer:
    """Central monitoring system for multiple bins"""
    
    def __init__(self):
        self.bins: Dict[str, SmartDustbin] = {}
    
    def add_bin(self, bin_id: str, capacity: float = 50) -> None:
        """Add new dustbin to monitoring system"""
        self.bins[bin_id] = SmartDustbin(bin_id, capacity)
        print(f"✅ Added {bin_id} to monitoring system")
    
    def monitor_all(self) -> None:
        """Monitor all bins simultaneously"""
        threads = []
        for bin_id, dustbin in self.bins.items():
            thread = threading.Thread(target=dustbin.run, name=f"Monitor-{bin_id}")
            threads.append(thread)
            thread.start()
        
        # Keep main thread alive
        try:
            while True:
                time.sleep(1)
        except KeyboardInterrupt:
            print("\n🛑 Stopping all bin monitoring...")
            for dustbin in self.bins.values():
                dustbin.is_running = False

def main():
    """Main function to run smart dustbin system"""
    print("🏭 SMART DUSTBIN MANAGEMENT SYSTEM")
    print("Version: 2.0 | IoT Enabled Waste Management")
    
    # Initialize monitoring server
    server = MonitoringServer()
    
    # Add multiple bins
    server.add_bin("BIN001", capacity=50)
    server.add_bin("BIN002", capacity=75)
    server.add_bin("BIN003", capacity=100)
    
    # Start monitoring all bins
    server.monitor_all()

if __name__ == "__main__":
    main()
