Sub extractNodesdata()
    Dim cntHpcRawData As Long
    Dim cntRow As Long
    Dim colSeq, colRawData, colDataTime, colType, colJobID, colGroup, colQueue, colRsrcNCPU, colRsrcNodes As Long
    Dim colQTime, colStart, colEnd, colExitStatus, colInQueue, colRunning, colExecVnode, colHpcNodes As Long
    Dim strTmp As String
    Dim valTime, posTmp1, posTmp2 As Long
    Dim timeQ, timeStart, timeEnd, timeQS, timeSE As Date
    
    Sheet3.Cells(3, 7) = "Step 3"
    Sheet3.Cells(3, 8) = Now
    
    cntHpcRawData = Sheet1.Range("B1000000").End(xlUp).Row
    colExecVnode = 16
    cntMaxNodes = 100
    
    cntNodeSection = 1
    
    colSeq = 1
    Sheet2.Cells(cntNodeSection, colSeq) = "Seq"
    colType = 2
    Sheet2.Cells(cntNodeSection, colType) = "Type"
    colJobID = 3
    Sheet2.Cells(cntNodeSection, colJobID) = "JobID"
    colGroup = 4
    Sheet2.Cells(cntNodeSection, colGroup) = "Group"
    colQueue = 5
    Sheet2.Cells(cntNodeSection, colQueue) = "Queue"
    colRsrcNCPU = 6
    Sheet2.Cells(cntNodeSection, colRsrcNCPU) = "Rsrc NCPU"
    colRsrcNodes = 7
    Sheet2.Cells(cntNodeSection, colRsrcNodes) = "Rsrc Nodes"
    
    colQTime = 8
    Sheet2.Cells(cntNodeSection, colQTime) = "qTime"
    colStart = 9
    Sheet2.Cells(cntNodeSection, colStart) = "Start"
    colEnd = 10
    Sheet2.Cells(cntNodeSection, colEnd) = "End"
    
    colExitStatus = 11
    Sheet2.Cells(cntNodeSection, colExitStatus) = "Exit_Status"
    colInQueue = 12
    Sheet2.Cells(cntNodeSection, colInQueue) = "InQueue"
    colRunning = 13
    Sheet2.Cells(cntNodeSection, colRunning) = "Running"
    colHpcNodes = 14
    Sheet2.Cells(cntNodeSection, colHpcNodes) = "HPC Node"
    colHpcNodesCore = 15
    Sheet2.Cells(cntNodeSection, colHpcNodesCore) = "Core"
    colHpcNodesMem = 16
    Sheet2.Cells(cntNodeSection, colHpcNodesMem) = "Mem"
    colHpcNodesGpu = 17
    Sheet2.Cells(cntNodeSection, colHpcNodesGpu) = "GPU"
            
    
    
    cntNodeTableRow = cntNodeSection
    cntCursor = 2
    
    For cntCursor = 2 To cntHpcRawData
   
        If (Sheet1.Cells(cntCursor, colType + 2) = "E") Then
 
            
            For colIndex = 1 To cntMaxNodes - 15
            
                strNodeInfo = Sheet1.Cells(cntCursor, colHpcNodesCore + colIndex + 1)
                
                If strNodeInfo = "" Then
                    Exit For
                Else
                            
                    cntNodeTableRow = cntNodeTableRow + 1
           
                    Sheet2.Cells(cntNodeTableRow, colSeq) = Sheet1.Cells(cntCursor, colSeq)
                    Sheet2.Cells(cntNodeTableRow, colType) = Sheet1.Cells(cntCursor, colType + 2)
                    Sheet2.Cells(cntNodeTableRow, colJobID) = Sheet1.Cells(cntCursor, colJobID + 2)
                    Sheet2.Cells(cntNodeTableRow, colGroup) = Sheet1.Cells(cntCursor, colGroup + 2)
                    Sheet2.Cells(cntNodeTableRow, colQueue) = Sheet1.Cells(cntCursor, colQueue + 2)
                    Sheet2.Cells(cntNodeTableRow, colRsrcNCPU) = Sheet1.Cells(cntCursor, colRsrcNCPU + 2)
                    Sheet2.Cells(cntNodeTableRow, colRsrcNodes) = Sheet1.Cells(cntCursor, colRsrcNodes + 2)
                    Sheet2.Cells(cntNodeTableRow, colStart) = Format(Sheet1.Cells(cntCursor, colStart + 2), "YYYY-MM-DD HH:MM:SS")
                    Sheet2.Cells(cntNodeTableRow, colEnd) = Format(Sheet1.Cells(cntCursor, colEnd + 2), "YYYY-MM-DD HH:MM:SS")
                    Sheet2.Cells(cntNodeTableRow, colExitStatus) = Sheet1.Cells(cntCursor, colExitStatus + 2)
                    Sheet2.Cells(cntNodeTableRow, colInQueue) = Sheet1.Cells(cntCursor, colInQueue + 2)
                    Sheet2.Cells(cntNodeTableRow, colRunning) = Sheet1.Cells(cntCursor, colRunning + 2)
                    
                    posNodeInfo1 = InStr(1, strNodeInfo, ":")
                    posNodeInfo2 = InStr(posNodeInfo1 + 1, strNodeInfo, ":")
                    
                    Sheet2.Cells(cntNodeTableRow, colHpcNodes) = Mid(strNodeInfo, 1, posNodeInfo1 - 1)
                    If posNodeInfo2 = 0 Then
                        Sheet2.Cells(cntNodeTableRow, colHpcNodesCore) = Mid(strNodeInfo, posNodeInfo1 + 1)
                    Else
                        Sheet2.Cells(cntNodeTableRow, colHpcNodesCore) = Mid(strNodeInfo, posNodeInfo1 + 1, posNodeInfo2 - posNodeInfo1 - 1)
                        
                        posNodeInfo3 = InStr(1, strNodeInfo, "mem=")
                        If posNodeInfo3 <> 0 Then
                            Sheet2.Cells(cntNodeTableRow, colHpcNodesMem) = Mid(strNodeInfo, posNodeInfo3)
                        End If
                        posNodeInfo4 = InStr(1, strNodeInfo, "ngpus=")
                        If posNodeInfo4 <> 0 Then
                            Sheet2.Cells(cntNodeTableRow, colHpcNodesGpu) = Mid(strNodeInfo, posNodeInfo4)
                        End If
                    End If
            
                End If
            
            Next colIndex

        End If
    
    Next cntCursor
        
    Sheet3.Cells(3, 9) = Now

End Sub
