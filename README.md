from machine import Pin, PWM
from time import sleep

# Πινάκια PWM για τους κινητήρες
GP2 = PWM(Pin(2))  # Κινητήρας 1 μπροστά
GP3 = PWM(Pin(3))  # Κινητήρας 1 πίσω
GP6 = PWM(Pin(6))  # Κινητήρας 2 μπροστά
GP7 = PWM(Pin(7))  # Κινητήρας 2 πίσω

# Συχνότητα PWM
freq = 1000
GP2.freq(freq)
GP3.freq(freq)
GP6.freq(freq)
GP7.freq(freq)

# Αισθητήρες γραμμής (0 = πάνω στη γραμμή, 1 = εκτός γραμμής)
left_sensor = Pin(0, Pin.IN)
middle_sensor = Pin(1, Pin.IN)
right_sensor = Pin(27, Pin.IN)

# Έλεγχος ταχύτητας
speed = 37000       # Ταχύτητα προς τα εμπρός
turn_speed = 30000  # Ταχύτητα στροφής
reverse_speed = 25000  # Ταχύτητα προς τα πίσω
reverse_time = 0.2  # Δευτερόλεπτα όπισθεν πριν ξαναγίνει έλεγχος

def line_tracking():
    """Ανάγνωση αισθητήρων με debounce"""
    sleep(0.01)
    return (left_sensor.value(), middle_sensor.value(), right_sensor.value())

def forward():
    """Κίνηση μπροστά"""
    GP2.duty_u16(speed)
    GP3.duty_u16(0)
    GP6.duty_u16(speed)
    GP7.duty_u16(0)

def backward():
    """Κίνηση προς τα πίσω"""
    GP2.duty_u16(0)
    GP3.duty_u16(reverse_speed)
    GP6.duty_u16(0)
    GP7.duty_u16(reverse_speed)

def turn_left():
    """Στροφή αριστερά"""
    GP2.duty_u16(0)
    GP3.duty_u16(turn_speed)
    GP6.duty_u16(turn_speed)
    GP7.duty_u16(0)

def turn_right():
    """Στροφή δεξιά"""
    GP2.duty_u16(turn_speed)
    GP3.duty_u16(0)
    GP6.duty_u16(0)
    GP7.duty_u16(turn_speed)

def stop():
    """Σταμάτημα κινητήρων"""
    GP2.duty_u16(0)
    GP3.duty_u16(0)
    GP6.duty_u16(0)
    GP7.duty_u16(0)

def handle_all_black():
    """Ειδικός χειρισμός όταν όλοι οι αισθητήρες βλέπουν μαύρο"""
    print("Όλοι οι αισθητήρες βλέπουν μαύρο - κάνω όπισθεν")
    backward()
    sleep(reverse_time)  # Όπισθεν για καθορισμένο χρόνο
    stop()
    
    # Έλεγχος αν επανήλθε στη γραμμή
    left, middle, right = line_tracking()
    if not (left == 0 and middle == 0 and right == 0):
        return True  # Επιτυχής επαναφορά
    return False  # Ακόμα όλοι οι αισθητήρες βλέπουν μαύρο

def robot_control():
    """Κύρια λογική ελέγχου"""
    left, middle, right = line_tracking()

    if left == 0 and middle == 0 and right == 0:
        if not handle_all_black():
            # Αν ακόμα βλέπει μαύρο μετά την όπισθεν, σταματά εντελώς
            stop()
            print("Πλήρες σταμάτημα - δεν ανιχνεύθηκε γραμμή")
    elif left == 0:
        turn_left()
    elif right == 0:
        turn_right()
    else:
        forward()

if __name__ == '__main__':
    try:
        while True:
            robot_control()
    except KeyboardInterrupt:
        stop()
        print("Το ρομπότ σταμάτησε από τον χρήστη")

