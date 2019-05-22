# Converting PowerShell objects to JavaScript objects with dynamic properties in vRealize Orchestrator

I've been working on a workflow that would complete all of our decommission steps for a Virtual Machine.  One step was to alert the DBAs if the VM has SQL on it.  So at this point, I am scanning all of our VMs and inventorying what has SQL, and what doesn't, and importing that into our CMDB.  I've taken this data, and I've used it to tag the VMs with the engine that is installed.  So I have a Tag Category called SQLEngine, and the Tag Value is the engine installed on that VM.  I figured the easiest way to figure out if a VM has SQL, in VRO, will be to read the tag assignments, and look for this flag.  The problem is we still have VCenters on v6.0, and don't have the VAPI available for our entire environment yet, which is when tagging is really fully functional from an API standpoint.  So in the mean time, any other tag operations I'm doing via VRO are being done with PowerShell via PowerShellHost.  I figured, I'd just do Get-Vm | Get-TagAssignment, and output that to VRO and I'd get the tag assignments that way - while I was at it, I'd go ahead and create a reusable action that just returned ALL tags, and not just the ones I'm interested in for this purpose - because code should be reusable! 

## As it turned out, this was much harder than anticipated.  

I already had an action for running PowerShell scripts, given that the output object attributes are known, and converting those to an object, which I copied mostly from this Rokett (https://rokett.github.io/post/convert-powershell-objects-to-javascript-objects-in-vrealize-orchestrator/).  But in this case, I have a dynamic list of tag categories that would be returned, so I don't know the attributes ahead of time.  So I set out to figure out a way to return a dynamic object from PowerShell, and convert that to a javascript object. Unfortunately, it doesn't seem that there is any way to get a list of attributes from the PowershellRemotePSObject. At least not that I could find.

## getXml(), RegEx

Finally, I settled on the idea that I could possibly output the object as XML, and then use some combination of getChildNodes() to get the properties, and then I could loop over the array of PowershellRemotePSObject, and grab the properties of each item, and create an object with the corresponding properties.  The problem here was that it seemed like I could get a unique list of attributes, but if, for instance, I was querying the tags of multiple Vms, the return objects would be heterogeneous, and wouldn't all have the same attributes.  So, then I looked at the XML, and realized I could probably parse it with RegEx.

## Helpful tool

If you've never used it, https://regex101.com/ has an online editor where you can feed it a RegExp, and an input string, and it will show you how it gets parsed by the interpreter.  This is a life saver when you're trying to build a regex.  So what I came up with is that there are basically two things I need to worry about: for each object returned by the output, there is a \<MS\> ... \</MS\> node.  Under that, each property is under \<S N="{attribute name}"\>{attribute value}\</S\> 

So I came up with two regular expressions that would fit these:

     /<MS>.*<\/MS>/g;
     /<S N=\"([^\"]+)\">([^<]+)<\/S>/g;

For the second, I use capture groups to grab the attribute names and values, and I just loop over the objects, and create a dictionary, looping over the sub-nodes containing the attributes.  Here is what the Action in VRO looks like: 


    //input parameter powershellHost(PowerShell:PowerShellHost)
    //input parameter script(string)

    var output = [];
    var session;
    try {
        session = powershellHost.openSession();
        var invokeResult = session.invokeScript(script);
        var psObject = invokeResult.getResults();
    } finally {
        if (session){
            powershellHost.closeSession(session.getSessionId());
        }
    }

    if ( psObject != null ) {
        var xml = psObject.getXml();
        var outerRegex = /<MS>.*<\/MS>/g;
        var outerMatch = outerRegex.exec(xml);

        while (outerMatch != null) {
            var temp = {};
            var inner = outerMatch[0];
            var regex = /<S N=\"([^\"]+)\">([^<]+)<\/S>/g;
            match = regex.exec(inner);

            while (match != null) {
                        // uncomment next line to see all available properties on the object(s)
                // System.log(match[1] + ":" + match[2]);
                temp[match[1]] = match[2];
                match = regex.exec(inner);
            }
            output.push(temp);
            var outerMatch = outerRegex.exec(xml);
        }
    }
    return output;
    
## And in case you're wondering what the script to grab the tags looks like:
    
    //input parameter powershellHost(PowerShell:PowerShellHost)
    //input parameter vm(VC:VirtualMachine)
    //input parameter VcUsername(string)
    //input parameter VcPassword(string)
    
    // Get the Sdk Connection from the vm object, and parse out the hostname/IP Address with regex
    var VcSdkConnection = vm.SdkConnection;
    var extractVcNameRegex = /^(?:https?:\/\/)?(?:[^@\n]+@)?(?:www\.)?([^:\/\n?]+)/im
    var matches_array = VcSdkConnection.name.match(extractVcNameRegex);
    var VCName = matches_array[1];
    
    // Get the VM Id to pass to Get-Vm
    var MoRef = "VirtualMachine-" + vm.Id
    // START GENERATING A DYNAMIC PS Script //
    // set up VCenter connection with Connect-ViServer
    var psScript = '$Server = "' + VCName + '"\n';
    
    // Connect to the ViServer - if UserId and Password were passed, use that, otherwise connect with SSO
    psScript += 'Connect-ViServer -Server $Server';
    if ( VcUsername && VcPassword ) { 
        psScript += ' -Username ' + VcUsername + ' -Password "' + VcPassword + '"';
    }
    psScript += '| Out-Null\n';

    // get tag assignments
    psScript += '$Vm = Get-Vm -Id ' + MoRef + ' -Server $Server\n';
    psScript += '$Tags = $Vm | Get-TagAssignment\n';

    // if no tag assignments, return null
    psScript += 'if ( $null -eq $Tags ) {\n';
    psScript += '    exit\n';
    psScript += '}\n';

    // Start creating the output object by adding the VirtualMachine Name and MoRef
    psScript += '$Output = [PSCustomObject]@{VirtualMachine = $Vm.Name;MoRef = "' + MoRef + '"}\n\n';
    psScript += 'ForEach ($TagAssignment in $Tags) {\n';
    psScript += '    $Category = $TagAssignment.Tag.Category.Name\n';
    psScript += '    $Tag = $TagAssignment.Tag.Name\n';
    psScript += '    if ($Output.PSObject.Properties.Name -contains $Category) {\n';
    // There may be multiple tags for a category, so I elected to put the iteration in brackets, e.g.:
    // Tag = "Myvalue1"
    // Tag[1] = "MyValue2"
    psScript += '        $Count = ($Output.PSObject.Properties.Name | Where-Object { $_ -match "$Category\[[0-9]+\]" }).Count + 1\n';
    psScript += '        $Category = "{0}[{1}]" -f $Category, $Count\n';
    psScript += '    }\n';
    psScript += '    $Output | Add-Member -MemberType NoteProperty -Name $Category -Value $Tag\n';
    psScript += '}\n';

    psScript += '$Output\n';

    // execute the script using the generic action for dynamic output from PS
    try {
      var tags = System.getModule("org.myorg.powershell").executeScriptOnPowershellHostDynamicOutput(powershellHost, psScript);
    } 
    catch (e){
      System.log("Error encountered executing PS script");
      System.log(e);
      var tags = null;
    }
    finally {
      return tags;
    }
