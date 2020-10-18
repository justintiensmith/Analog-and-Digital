# Circuit Playground Sound Meter

import array
import math
import audiobusio
import board
import neopixel
import random

# Number of total pixels - 10 build into Circuit Playground
NUM_PIXELS = 10

# Exponential scaling factor.
# Should probably be in range -10 .. 10 to be reasonable.
CURVE = 4
SCALE_EXPONENT = math.pow(10, CURVE * -0.05)

# Number of samples to read at once.
NUM_SAMPLES = 160

# Restrict value to be between floor and ceiling.
def constrain(value, floor, ceiling):
    return max(floor, min(value, ceiling))


# Scale input_value between output_min and output_max, exponentially.
def log_scale(input_value, input_min, input_max, output_min, output_max):
    normalized_input_value = (input_value - input_min) / \
                             (input_max - input_min)
    return output_min + \
        math.pow(normalized_input_value, SCALE_EXPONENT) \
        * (output_max - output_min)


# Remove DC bias before computing RMS.
def normalized_rms(values):
    minbuf = int(mean(values))
    samples_sum = sum(
        float(sample - minbuf) * (sample - minbuf)
        for sample in values
    )

    return math.sqrt(samples_sum / len(values))


def mean(values):
    return sum(values) / len(values)


# Main program

# Set up NeoPixels and turn them all off.
pixels = neopixel.NeoPixel(board.NEOPIXEL, NUM_PIXELS, brightness=0.1, auto_write=False)
pixels.fill(0)
pixels.show()

mic = audiobusio.PDMIn(board.MICROPHONE_CLOCK, board.MICROPHONE_DATA,
                       sample_rate=16000, bit_depth=16)

# Record an initial sample to calibrate. Assume it's quiet when we start.
samples = array.array('H', [0] * NUM_SAMPLES)
mic.record(samples, len(samples))
# Set lowest level to expect, plus a little.
#input_floor = normalized_rms(samples) + 10
# OR: used a fixed floor
input_floor = 100

# You might want to print the input_floor to help adjust other values.
# print(input_floor)

# Corresponds to sensitivity: lower means more pixels light up with lower sound
# Adjust this as you see fit.
input_ceiling = input_floor + 800

peak = 0
while True:
    mic.record(samples, len(samples))
    magnitude = normalized_rms(samples)
    # You might want to print this to see the values.
    # print(magnitude)

    # Compute scaled logarithmic reading in the range 0 to NUM_PIXELS
    c = log_scale(constrain(magnitude, input_floor, input_ceiling),
                  input_floor, input_ceiling, 0, NUM_PIXELS)
    print((c,))
    # Light up pixels that are below the scaled and interpolated magnitude.
    pixels.fill(0)
    c=c*25
    COLOR=(int(c),int(c),int(c))
    for i in range(NUM_PIXELS):
        pixels[i]=COLOR
    pixels.show()

RED = (255, 10, 0)
ORANGE = (255, 100, 0)
YELLOW = (255, 247, 0)
GREEN = (72, 255, 0)
AQUA = (0, 255, 221)
BLUE = (0, 76, 255)
INDIGO = (255, 0, 170)
VIOLET = (180, 0, 255)

colors [RED, ORANGE, YELLOW, GREEN, AQUA, BLUE, INDIGO, VIOLET]


