<!--
NavPath: Face API/How-to Topics
LinkLabel: How to Add Faces
Url: face-api/documentation/face-api-how-to-topics/howtoaddfaces
Weight: 220
-->
# How to Add Faces

This guide will demonstrate the best practice to add massive number of persons and faces to a person group, which also applies to face lists. The samples are written in C# using the Face API client library.

## Table of Contents

- [Step 1: Initialization](#step1)
- [Step 1: Authorize the API call](#step1)
- [Step 2: Create the person group](#step2)
- [Step 3: Create the persons to the person group](#step3)
- [Step 4: Add faces to the persons](#step4)
- [Summary](#summary)

## <a name="step1"></a> Step 1: Initialization

Several variables are declared and a helper function is implemented to schedule the requests.

- `PersonCount` is the toal number of persons.
- `CallLimitPerSecond` is the maximum calls per second according to the subscription.
- `_timeStampQueue` is a Queue to record the request timestamps.
- `await WaitCallLimitPerSecond()` will wait until it is valid to send next reqeust.

```CSharp
const int PersonCount = 10000;
const int CallLimitPerSecond = 10;
static Queue<DateTime> _timeStampQueue = new Queue<DateTime>(CallLimitPerSecond);

static async Task WaitCallLimitPerSecond()
{
    Monitor.Enter(_timeStampQueue);
    try
    {
        if (_timeStampQueue.Count >= CallLimitPerSecond)
        {
            TimeSpan timeInterval = DateTime.UtcNow - _timeStampQueue.Peek();
            if (timeInterval < TimeSpan.FromSeconds(1))
            {
                await Task.Delay(TimeSpan.FromSeconds(1) - timeInterval);
            }
            _timeStampQueue.Dequeue();
        }
        _timeStampQueue.Enqueue(DateTime.UtcNow);
    }
    finally
    {
        Monitor.Exit(_timeStampQueue);
    }
}
```

## <a name="step2"></a> Step 2: Authorize the API call

When using a client library, the subscription key is passed in through the constructor of the FaceServiceClient class. For example:

```CSharp
FaceServiceClient faceServiceClient = new FaceServiceClient("subscription key");
```

The subscription key can be obtained from the Marketplace page of your Azure management portal. See [Subscriptions](https://www.microsoft.com/cognitive-services/en-us/sign-up).

## <a name="step3"></a> Step 3: Create the person group

A person group named "MyPersonGroup" is create to save the persons.
Note that we also enqueue the request time to `_timeStampQueue` to ensure the overall functionality.

```CSharp
const string personGroupId = "mypersongroupid";
const string personGroupName = "MyPersonGroup";
_timeStampQueue.Enqueue(DateTime.UtcNow);
await faceServiceClient.CreatePersonGroupAsync(personGroupId, personGroupName);
```

## <a name="step4"></a> Step 4: Create the persons to the person group

Persons are created concurrently and `await WaitCallLimitPerSecond()` is also applied to avoid exceeding the call limit.

```CSharp
CreatePersonResult[] persons = new CreatePersonResult[PersonCount];
Parallel.For(0, PersonCount, async i =>
{
    await WaitCallLimitPerSecond();

    string personName = $"PersonName#{i}";
    persons[i] = await faceServiceClient.CreatePersonAsync(personGroupId, personName);
});
```

## <a name="step5"></a> Step 5: Add faces to the persons

Adding faces to different persons are processed concurrently, while it is recommended to add faces to one specific person sequentially.
Again, `await WaitCallLimitPerSecond()` is invoked to ensure the reqeust frequency.

```CSharp
Parallel.For(0, PersonCount, async i =>
{
    Guid personId = persons[i].PersonId;
    string personImageDir = @"/path/to/person/i/images";

    foreach (string imagePath in Directory.GetFiles(personImageDir, "*.jpg"))
    {
        await WaitCallLimitPerSecond();

        using (Stream stream = File.OpenRead(imagePath))
        {
            await faceServiceClient.AddPersonFaceAsync(personGroupId, personId, stream);
        }
    }
});
```

## <a name="summary"></a> Summary

In this guide you have learned the process of creating a person group with massive number of persons and faces. Several reminders:

- This strategy also applies to add faces to face lists, which is adviced to add face to different face lists concurrently, and same operation to one specific should be done sequentially.
- To keep the simplicity, the handling of potential exception is omitted in this guide. If you want to enhance more robustness, proper retry policy should be applied.

The following are a quick reminder of the features previously explained and demonstrated:

- Creating person groups using the [Person Group - Create a Person Group](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) API
- Creating persons using the [Person - Create a Person](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c) API
- Adding faces to persons using the [Person - Add a Person Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) API
