---
layout: post
title: "Sign flips to opposite"
date: 2021-07-04T16:03:13+02:00
tags: excel VBA
---
# Problem

I have a data (negative, positive and null values) stored in rows per each column. I wanted to to find out via usual Excel formulas how could I tell when exactly was the last change of the value store in the row- from negative to positive or null and vice versa.
I didn't manage to solve it via formulas unfortunately. So I wrote below VBA UDF.

# Solution

My idea was to loop from the right to left (within some range) and compare current cell sign with previous cell sign and based on that observation tell when the sign swapped- like two periods ago, 5 periods ago or never (say we give *0* if no change ever).
I applied Sgn function to return the value of the cell (1,0 or -1). 

We can imagine data to be stored in this way:

|1  | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | Jan | Feb | Mar | Apr | May | Jun | Result |
|---|---|---|---|---|---|---|---|---|-----|-----|-----|-----|-----|-----|--------|
|2  |aaa|bbb|ccc|ddd|eee|fff|ggg|hhh| -10 | 10  | 20  | -50 |  0  | 1   | 1
|3  |iii|jjj|kkk|lll|mmm|nnn|ooo|ppp| -10 | -1  | 20  | 50  |  20 | 1   | 5
|4  |zzz|aaa|uuu|rrr|fff|vvv|lll|qqq| 10  |  1  | 20  | 50  |  20 | 1   | 0

* row 2, May period got 0 and June 1- that's change of 1 period.
* row 3, Feb was negative so since June period it's 5 periods since last sign change.
* row 4, from Jan till Jun every value was positive, therefore we set *0* as the result to distinct this very case.

# Code
```visualbasic

    Option Explicit
    Option Base 1

    Function Sign_Change(Cell_Sign As Range) As Integer

    Dim i As Integer
    Dim c As Integer
    Dim j As Integer
    Dim Rng As Integer
    Dim Temp() As Integer
    Dim Result() As Integer
    Dim Data As Worksheet
    Dim Offset_Range As Integer

    Set Data = Worksheets("Data")

    'Set off the range number of columns left most+2; otherwise use "On Error Resume 0"
    
    Offset_Range = 8
    Rng = Range_Limit(1) - 1
    ReDim Preserve Temp(1 To Rng)
    
    'Loop in the range and collect value in Temp in case the sign in cell has changed;
    'if the previous cell sign is equal current cell sign give it 0 otherwise store the value from the range 
    
    For i = Rng To Offset_Range Step -1

        If Sgn(Data.Cells(Cell_Sign.Row, i - 1)) <> Sgn(Data.Cells(Cell_Sign.Row, i)) Then
            i = i - 1
            c = Rng - i
            Temp(i) = c
                Else
                    c = vbNull
        End If
    
    Next i

    'Loop in the range of the Temp and assign only value different then 0 in new array Period.
    'This is in order to collect only values where the sign changed.
    'In the end take the maximum value of the array Result- this is what we have been looking for.
    
    For j = LBound(Temp) To UBound(Temp)

        If Temp(j) <> 0 Then
            ReDim Result(j)
                Result(j) = Temp(j)
        End If
    
    Next j

    On Error Resume Next

    Sign_Change = Application.Max(Result)

    End Function

    'This function returns number of columns.
    
    Function Return_LC(row_index As Integer) As Integer

    Dim Data As Worksheet
    Set Data = Worksheets("Data")

        Return_LC = Data.Cells(row_index, Columns.Count).End(xlToLeft).Column

    End Function

    'This function returns range where the function sign_change will perform look up.
    'Its base start cell is set to the column header "Result" in this example.
    
    Function Range_Limit(row_index As Integer) As Integer

    Dim i As Integer
    Dim j As Integer

    Dim Data As Worksheet
    Set Data = Worksheets("Data")

    For i = 1 To 200

        If Data.Cells(row_index, i).Value = "Result" Then
           i = i + 1
           j = i - 1
        End If
    Next i

    Range_Limit = j
    End Function
```
