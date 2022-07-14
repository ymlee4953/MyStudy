Sub DataInitial()
    Dim cntHpcRawData As Long
    Dim cntRow As Long
    Dim colSeq, colRawData, colDataTime, colType, colJobID, colGroup, colQueue, colRsrcNCPU, colRsrcNodes As Long
    Dim colQTime, colStart, colEnd, colExitStatus, colInQueue, colRunning, colExecVnode As Long
    Dim strTmp As String
    Dim valTime, posTmp1, posTmp2 As Long
    Dim timeQ, timeStart, timeEnd, timeQS, timeSE As Date
    
    Sheet3.Cells(1, 7) = "Step 1"
    Sheet3.Cells(1, 8) = Now
    
    colSeq = 1
    Sheet1.Cells(1, colSeq) = "Seq"
    colRawData = 2
    Sheet1.Cells(1, colRawData) = "RawData"
    colDataTime = 3
    Sheet1.Cells(1, colDataTime) = "DataTime"
    colType = 4
    Sheet1.Cells(1, colType) = "Type"
    colJobID = 5
    Sheet1.Cells(1, colJobID) = "JobID"
    colGroup = 6
    Sheet1.Cells(1, colGroup) = "Group"
    colQueue = 7
    Sheet1.Cells(1, colQueue) = "Queue"
    colRsrcNCPU = 8
    Sheet1.Cells(1, colRsrcNCPU) = "Rsrc NCPU"
    colRsrcNodes = 9
    Sheet1.Cells(1, colRsrcNodes) = "Rsrc Nodes"
    
    colQTime = 10
    Sheet1.Cells(1, colQTime) = "qTime"
    colStart = 11
    Sheet1.Cells(1, colStart) = "Start"
    colEnd = 12
    Sheet1.Cells(1, colEnd) = "End"
    
    colExitStatus = 13
    Sheet1.Cells(1, colExitStatus) = "Exit_Status"
    colInQueue = 14
    Sheet1.Cells(1, colInQueue) = "InQueue"
    colRunning = 15
    Sheet1.Cells(1, colRunning) = "Running"
    
    colExecVnode = 16
    Sheet1.Cells(1, colExecVnode) = "Exec_Vnodes"
    
    cntHpcRawData = Sheet1.Range("B1000000").End(xlUp).Row
    
    For cntRow = 2 To cntHpcRawData
        Sheet1.Cells(cntRow, colSeq) = cntRow - 1
        Sheet1.Cells(cntRow, colDataTime) = Mid(Sheet1.Cells(cntRow, colRawData), 1, 19)
        Sheet1.Cells(cntRow, colType) = Mid(Sheet1.Cells(cntRow, colRawData), 21, 1)
        
        If Sheet1.Cells(cntRow, colType) = "E" Then
        
            Sheet1.Cells(cntRow, colJobID) = Mid(Sheet1.Cells(cntRow, colRawData), 23, 6)
        
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "group=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 6, 50)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colGroup) = Mid(strTmp, 1, posTmp2 - 1)
        
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "queue=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 6, 50)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colQueue) = Mid(strTmp, 1, posTmp2 - 1)
                    
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "Resource_List.ncpus=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 20, 50)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colRsrcNCPU) = Mid(strTmp, 1, posTmp2 - 1)
                                  
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "Resource_List.nodect=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 21, 50)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colRsrcNodes) = Mid(strTmp, 1, posTmp2 - 1)
                     
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "qtime=")
            valTime = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 6, 10)
            Sheet1.Cells(cntRow, colQTime) = Format((valTime / 86400) + DateSerial(1970, 1, 1) + TimeSerial(9, 0, 0), "yyyy-mm-dd hh:mm:ss")
          
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "start=")
            valTime = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 6, 10)
            Sheet1.Cells(cntRow, colStart) = Format((valTime / 86400) + DateSerial(1970, 1, 1) + TimeSerial(9, 0, 0), "yyyy-mm-dd hh:mm:ss")
          
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "end=")
            valTime = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 4, 10)
            Sheet1.Cells(cntRow, colEnd) = Format((valTime / 86400) + DateSerial(1970, 1, 1) + TimeSerial(9, 0, 0), "yyyy-mm-dd hh:mm:ss")
          
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "Exit_status=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 12, 50)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colExitStatus) = Mid(strTmp, 1, posTmp2 - 1)
            
            tmFrom = Sheet1.Cells(cntRow, colQTime)
            tmTo = Sheet1.Cells(cntRow, colStart)
                        
            tmDiff = Format(tmTo - tmFrom, "HH:MM:SS")
            dtDiff = DateDiff("d", tmFrom, tmTo)
            
            numTmFrom = Hour(tmFrom) * 10000 + Minute(tmFrom) * 100 + Second(tmFrom)
            numTmTo = Hour(tmTo) * 10000 + Minute(tmTo) * 100 + Second(tmTo)
            
            If numTmTo >= numTmFrom Then
                If dtDiff > 0 Then
                    Sheet1.Cells(cntRow, colInQueue) = dtDiff & "d " & tmDiff
                Else
                    Sheet1.Cells(cntRow, colInQueue) = "  " & tmDiff
                End If
            Else
                dtDiff = dtDiff - 1
                If dtDiff > 0 Then
                    Sheet1.Cells(cntRow, colInQueue) = dtDiff & "d " & tmDiff
                Else
                    Sheet1.Cells(cntRow, colInQueue) = "  " & tmDiff
                End If
            End If
            
            tmFrom = Sheet1.Cells(cntRow, colStart)
            tmTo = Sheet1.Cells(cntRow, colEnd)
                        
            tmDiff = Format(tmTo - tmFrom, "HH:MM:SS")
            dtDiff = DateDiff("d", tmFrom, tmTo)
            
            numTmFrom = Hour(tmFrom) * 10000 + Minute(tmFrom) * 100 + Second(tmFrom)
            numTmTo = Hour(tmTo) * 10000 + Minute(tmTo) * 100 + Second(tmTo)
            
            If numTmTo >= numTmFrom Then
                If dtDiff > 0 Then
                    Sheet1.Cells(cntRow, colRunning) = dtDiff & "d " & tmDiff
                Else
                    Sheet1.Cells(cntRow, colRunning) = "  " & tmDiff
                End If
            Else
                dtDiff = dtDiff - 1
                If dtDiff > 0 Then
                    Sheet1.Cells(cntRow, colRunning) = dtDiff & "d " & tmDiff
                Else
                    Sheet1.Cells(cntRow, colRunning) = "  " & tmDiff
                End If
            End If
            
            posTmp1 = InStr(Sheet1.Cells(cntRow, colRawData), "exec_vnode=")
            strTmp = Mid(Sheet1.Cells(cntRow, colRawData), posTmp1 + 11, 1000)
            posTmp2 = InStr(strTmp, " ")
            Sheet1.Cells(cntRow, colExecVnode) = Mid(strTmp, 1, posTmp2 - 1)
                              
        End If
        
    Next cntRow
   
    Sheet3.Cells(1, 9) = Now
    
End Sub

