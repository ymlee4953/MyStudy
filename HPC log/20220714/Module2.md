Sub splitUpNodesData()
    Dim strNodesData As String
    Dim colExecVnode As Long
    Dim cntNodes As Long
    
    Dim cntHpcRawData As Long
    Dim cntRow As Long
    Dim cntMaxNodes As Long
    Dim posNodesFrom, posNodesTo As Long
        
    Sheet3.Cells(2, 7) = "Step 2"
    Sheet3.Cells(2, 8) = Now
    
    cntHpcRawData = Sheet1.Range("B1000000").End(xlUp).Row
    colExecVnode = 16
    cntMaxNodes = 1000
        
    For cntRow = 2 To cntHpcRawData
        strNodesData = Sheet1.Cells(cntRow, colExecVnode)
        If strNodesData = "" Then
            Exit For
        End If
        
        posNodesFrom = InStr(1, strNodesData, "(")
        
        For cntNodes = 1 To cntMaxNodes
            posNodesTo = InStr(posNodesFrom, strNodesData, ")")
            
            Sheet1.Cells(cntRow, colExecVnode + cntNodes) = Mid(strNodesData, posNodesFrom + 1, posNodesTo - posNodesFrom - 1)
            posNodesFrom = InStr(posNodesTo, strNodesData, "(")
            
            If Sheet1.Cells(1, colExecVnode + cntNodes) = "" Then
                Sheet1.Cells(1, colExecVnode + cntNodes) = "No.Nodes" & Format(cntNodes, "000")
            End If
            
            If posNodesFrom = 0 Then
                Exit For
            End If
        
        Next cntNodes
    Next cntRow
    
    Sheet3.Cells(2, 9) = Now    
    
End Sub


