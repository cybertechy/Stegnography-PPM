# Stegnography PPM P3 files

This C program performs stegnography for Plain PPM images (P3).

## Data Structures

Some of the data structure used include a singly linked list. The list was used to store the comments. The linked list is formed of Nodes which contains an character pointer called comment, containing one line of comment.

Each line of comment will form a new node and the PPM structure contains the head node called comments.

Other structures Include the Array used for storing the Pixels and its values. The Array is declared as a pointer to collection of Pixel structures. The Size of the Array is the Total no.of pixels (WxH) into size of the Pixel structure. More Info at Section 1.1.
  

## Design
The File that performs Stegnography is: `steg.c` 
Compiling this file can be performed in two ways:

 1. Tradional Way: `gcc -o steg steg.c -lm`
 2. Easier Way: 	`make`
 
Note:
The file steg.c uses additional standard libraries: 
 1. math.h 
 2. stdint.h 
 3. ctype.h
 
 
 
To use the math.h file the code we have to link the files using the `-lm` command

To ensure this in the make file also, the code has been changed to:

    # Options to pass to the C compiler
    CFLAGS  = -std=c99 -O2 -Wall -pedantic -g
    LDLIBS  = -lm
    # List of programs to build
    TARGETS  = steg
    # Default rule, built if you just say "make"
    all:  $(TARGETS)
    # Build a program
    %:  %.c
    $(CC)  $(CFLAGS) -o $@  $< -lm
    # Remove built programs and temporary files
    clean:
      rm -f $(TARGETS) *~

The math.h header has been linked so as functions such as log2(), log10(), pow() can be used easily.

The purpose of the stdint.h is to have more specific data types for integers used it the program, more on it at 1.1

The ctype.h Library was used to ensure only ascii character are used when encoding the message.

# 1.1 Structures

The file contains three structs

1. struct PPM
2. struct Pixel
3. struct Node

The struct PPM stores Information about the PPM image file
and contains the following fields:

 1. `format` - A character array of size 3 storing the file format ex. "P3"
 2. Node pointer `comments` - points to Node thats stores comments in the file
 3. `width` - width specified in the file
 4. `height`- height specified in the file
 5. unsigned 16 bit int `max` - max value of colour
 6. Pointer to struct pixel `pixels` - array of pixels

The struct Pixel contains:

 1. unsigned int 16bit `red` - value of red pixel
 2. unsigned int 16bit `green` - value of green pixel
 3. unsigned int 16bit `blue` - value of blue pixel

Originally the data type was **int** , but since as per the PPM format the max colour value must be less than 65536 , To make it **resource efficient** the data type allocation is changed from **4 Bytes to 2 Bytes** using the uint16_t of stdint library which increases efficiency by **50%**.

**(Additional PPM Features)**
The Node struct implements storing of the comments as a linked list of strings. It contains the following field:

 1. `comment` - pointer to array of characters (String)
 2. `next` - pointer to the next Node containing comments.


## 1.2 Prepocessor Definitions

There are two definitions defined which provide ease of creating the PPM and Pixel

1. `_PPM_` - struct PPM
2. `_Pixel_` - struct Pixel

## 1.3 Auxilary Functions:

Functions that are assisting in the main flow are:

 1. `exception(char  *messaage)` - Prints the Message as Error and exits the code execution
 2. `comments_retriever(FILE  *f,  struct  Node  *head)` - reads the comments from the file and stores linked nodes to the head pointer
 3. `garbage_collector(_PPM_  *img)` - frees the dynamic memory allocation of the ppm struct and its components
 4. `countBits(unsigned  int  num)` - returns the no.of bits required to store the number
 5. `encryptBit(struct  Pixel  *pixel,  int  bit)` - executes the algorithm of storing a bit into the pixel

## 2.1 Algorithms 

To encode a message into the PPM image this program use two algorithms.

1. **Reservation Circulation Comparison (RCC) Model**
This is a message distribution algorithm that deals with where the message should be stored in the Image. This algorithm is an extension of the spread specturm stegnography method. 

The spread spectrum method tells that the the message to be hidden must be scattered accross the entire cover Image. The RCC model ensures that each bit of information is spread at equal distances from each other. This was calculated using the formula:

`Distance = Total no.of Pixels / No.of Pixels required by the Message`

The message that has to be stored is broken up to bits. A random Pixel is chosen using the `rand()` function which is seeded with time.
From the random pixel onwards the message bits are stored at equal distances from each other.

**Reservation**
Since the message bits are stored from random pixel onwards, the random pixel number becomes crucial when decoding the message as it helps identify the start of the sequence of message bits. So this random Pixel number is stored in the start of the image pixels. This part where pixel number is stored are called the reserved pixels and they are determined by the no.of bits that required to store the number (total no.of pixels) in the image.

Ex: If an Image has total of 16 pixels then to represent the number 16 it would require 5 bits. So then in the Image the first 5 pixels will be reserved. In this reserved pixels the random pixel number will be written.

**Circulation**
The message bits start from random pixel  and may go till the last pixel but if the message bits require more pixels they continue spreading evenly from the start of the pixels ie, The message bits wrap arround the image pixels. A simple analogy to explain this would be a Circular Linked List or Wrap Arround Queue.

**Comparison**
Since once a message is encoded there is no data of how long the message was ie. making it impossible for the decoder to find the distance between each bits. To overcome this the original image would be required to compare the values with the encoded cover image and get the bits.

2. **Least Significant Bit Technique (LSB)**
This Algorithm specifies how the message bits are written into the pixel. For this the right most bit of the Red Pixel is modified to store the message bit. 

Ex:
Red Pixel: 128 
Message Bit: 1

1000 0000 | 1 -> 1000 0001


*what happens if a letter you're going to hide has the same value as the pixel component it's going to replace?* 

Two Ensure that it gets detected in the decoding Algorithm when the message bit and red pixel bit is same the red pixel's second least significant Bit is Inverted.

Ex:   
Red Pixel: 255
Message Bit: 1


Step1: Invert the red pixel second least significant bit
1111 1111  -> 1111 1101

step2: write message bit into the LSB
1111 1101 | 1 -> 1111 1101

This Method Esures the difference between the orginal and modified values are no greater than + or - 2. Ensuring no great difference in colours are observed.

## Encoding Features
When encoding the program calculates the Possible no.of Characters that can be encoded and then prompts to Enter the Message. Based upon the length of characters it calculates the load factor.
Once Encoding is complete the program computes the Mean Square Value (MSE) which is used to further compute the Peak Signal To Noise Ratio (PSNR) value in dB. The measure of how well the message has been hidden in the message is indicated by the PSNR value.

The Standard Range of PSNR value is from 30 - 50 dB. If the PSNR value is less than 30dB the program informs that the hidden message could be easily detected and recommends to use a bigger Image.

Reference: https://core.ac.uk/reader/81148270 (Result and Analysis)
