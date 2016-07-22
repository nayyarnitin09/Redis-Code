# Redis-CodeImports ServiceStack.Redis
Imports Jil
Imports Newtonsoft.Json
Imports System.IO


Public Class RedisInstance
    Dim host As String
    Dim port As Integer
    Dim rc As RedisClient
    Sub New()
        'host = "192.168.2.118"
        'port = 6749
        host = "localhost"
        port = 6379
        rc = New RedisClient(host, port)
    End Sub

    Public Sub ConnectToRedis()

    End Sub
    Public Function CheckRedisKey(ByVal redisKey As String) As Boolean
        If rc.Get(redisKey) Is Nothing Then
            Return False
        Else
            Return True
        End If
    End Function

    Public Sub SetKey(ByVal keyName As String, ByVal keyValue As String)
        rc.Set(keyName, keyValue)
        rc.Expire(keyName, 86400)
    End Sub

    Public Function Serialize(ByVal objToSearialize) As String
        Return JsonConvert.SerializeObject(objToSearialize)
    End Function

    Public Function GetKey(ByVal keyName As String) As String
        Return System.Text.Encoding.UTF8.GetString(rc.Get(keyName))
    End Function

    Public Function Deserialize(Of T)(ByVal serializedKey As String) As T
        Return JsonConvert.DeserializeObject(Of T)(serializedKey)
    End Function

    Public Sub FlushAll()
        rc.FlushAll()
    End Sub

    Public Sub DeleteKeys(ByVal pattern As String)
        For Each x In rc.Keys(pattern)
            rc.Del(x)
        Next
    End Sub

End Class

//Example of Calling above methods


  If Not ri.CheckRedisKey(keyRedis) Then
            dt = DatabaseClasses.DatabaseConnect.GetDataTable("Items_SimpleSearch_ALL", paramrry)
            serializedString = ri.Serialize(dt)
            ri.SetKey(keyRedis, serializedString)
            dt = FilterDataTableRow(dt, startRowIndex, endRowIndex)
        Else
            serializedString = ri.GetKey(keyRedis)
            deserializedObject = ri.Deserialize(Of String)(serializedString)
            dt = ri.Deserialize(Of DataTable)(deserializedObject)
            filterTable = dt.Clone()
            Dim dt1 As DataRow() = dt.Select("ShortDesc Like '%" & SearchString & "%' OR LongDesc Like '%" & SearchString & "%' OR ItemName Like '%" & SearchString & "%'  OR ItemNumber Like '%" & SearchString & "%' OR ManufacturerPartNumber Like '%" & SearchString & "%' OR Keywords Like '%" & SearchString & "%'")
            For Each dr As DataRow In dt1
                dr("NewRowNo") = i
                filterTable.Rows.Add(dr.ItemArray)
                i = i + 1
            Next

            If filterTable.Rows.Count > 0 Then
                dt = FilterDataTableRow(filterTable, startRowIndex, endRowIndex)
            End If
        End If
        Return dt
    End Function
