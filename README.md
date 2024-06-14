# Read/Write Direct-Access Binary Files in Fortran

Programmer : Kosei Ohara  
Last Update : 2024/04/26

## Test Environment
ifort 19.1.3.304 20200925

## About These Codes
These are the modules to read/write direct-access files in Fortran.  
NaNs are detected before output and after input.  
Functions are available for scalar, 1-dim, 2-dim, and 3-dim variables that are single or double precision.

## Structure
Exclusive structure 'finfo' is defined in fileio.f90 to read or write binary files efficiently.  
It has 6 parameters as explained below.

### unit
4 byte integer.  
The unit number to access files

### fname
character.  
Name of the file you want to open.  
Maximum length of the name is 128 characters.

### action
character.
Action to the opened file.  
"read", "write", and "readwrite" are available.  
Maximum length is 16 characters.

### recl
4 byte integer.  
The record length of each record.

### record
4 byte integer.  
The record to access.

### recstep
4 byte integer.  
record is updated after data is inputted/outputted with the function this module provides.  
recstep is the value to add to record in each input/output.

## Subroutines

### fopen
This subroutine defines all elements of finfo and opens file.  
Arguments are described below

#### ftype
finfo type structure.  
intent(out) attribute.

#### unit
4 byte integer.  
intent(in), optional attribute.  
If input is 5 or 6, this subroutine returns error stop.  
If unit is specified, the file is opened by the value.  
If not, the file is opened by the NEWUNIT option.
This value is substituted to ftype%unit.

#### fname
character.  
intent(in) attribute.  
The file name of what is opened.  
This string is substituted to ftype%fname.

#### action
character.  
intent(in) attribute.  
The action to the opened file.  
This string is substituted to ftype%action.

#### recl
4 byte integer.  
intent(in) attribute.  
The record length of each record.  
If the input is 0 or less, error is returned.  
This value is substituted to ftype%recl.

#### record
4 byte integer.  
intent(in) attribute.  
The record to access at the first step.  
If the input is 0 or less, error is returned.  
This value is substituted to ftype%record.

#### recstep
4 byte integer.  
intent(in) attribute.  
record is updated after data is inputted/outputted with the function this module provides.  
recstep is the value to add to record in each input/output.  
If the input is less than 0, error is returned.  
This value is substituted to ftype%recstep.

### fclose
This subroutine closes the file and rewrite the finfo structure.  
Arguments are described below

#### ftype
finfo type structure.  
intent(inout) attribute.  
0 is substituted to ftype%unit, ftype%recl, ftype%record, and ftype%recstep and "ERROR" to ftype%fname and ftype%action.

### fread
This subroutine reads the file and check the existence of NaN.  
Arguments are described below

#### ftype
finfo type structure.  
intent(inout) attribute.  
ftype%record is updated by ftype%recstep.

#### carrier
real.  
intent(out) attribute.  
single and double precision are available.  
Scalar, 1 dimension, 2 dimension, and 3 dimention array can be used for the argument.

### fwrite
This subroutine writes dat to the file and check the existence of NaN.  
Arguments are described below

#### ftype
finfo type structure.  
intent(inout) attribute.  
ftype%record is updated by ftype%recstep.

#### carrier
real.  
intent(in) attribute.  
single and double precision are available.  
Scalar, 1 dimension, 2 dimension, and 3 dimention array can be used for the argument.

### reset_record
This subroutine resets the record number to access in the next fread/fwrite.  
One of two options can be choosen : one is to substitute a specific value and the other is to add a specific value to the present record.  
Arguments are described below

#### ftype
finfo type structure.  
intent(inout) attribute.  
ftype%record is resetted in this subroutine.

#### newrecord
4 byte integer.  
intent(in), optional attribute.  
If this argument presents, ths value is substituted to ftype%record.

#### recstep
4 byte integer.  
intent(in), optional attribute.  
If this argument presents and newrecord does not present, ths value is added to ftype%record.

### get_record
This function returns the present record.  
Argument is described below

#### ftype
finfo type structure.  
intent(in) attribute.  

#### result
4 byte integer.  
ftype%record is returned.







