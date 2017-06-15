# Introduction to vRO Scriptable Tasks

For a few years now, PowerCLI has been my go-to automation tool for my VMWare environment.  A while back, I created a Windows Forms GUI, that did some really basic operations, but was used to give some advanced PowerCLI scripting to some end users that had no interest in running scripts.  So my VMManager was formed. Most of the functions I made available were already available via the GUI, but doing the same action across multiple VMs required scripting.  So I made a very basic text box, where a user could paste in a list of machines, and for example, initiate a VM Tools upgrade, or VM Hardware upgrade.  Selecting a checkbox would allow these users to restart an entire group of VMs, get a report of tools and hardware, or even take snapshots prior to doing maintenance.  

I started working with vRO when I realized that many of the same functions could be accessible from a right-click context inside of the VMWare client.  However, that still brought me back to only one machine at a time operations. I knew there had to be something better.  Now I am using the vRO REST API to submit requests for the functionality I built.  

The first script I decided to set out to write was a Scriptable Task that would take a string input for a VM Name, and find the associated VC:VirtualMachine object in the vCenter inventory.  This would allow me to build my workflows to accept simple types (Date, number, string), and work with complex objects.

The script object takes an input parameter (VMName) of type string, and it creates an output parameter (vm) of type VC:VirtualMachine.

## Here is the code:

	vCenters = VcPlugin.allSdkConnections;
	var vms = new Array();

	for each ( vCenter in vCenters ) {
	  System.log(vCenter.name);
	  var allVms = vCenter.getAllVirtualMachines();
	  for each (vm in allVms) {
		if ( vm.name == VMName) {
		  System.log("VM " + vm.name + " found on " + vCenter.name);
		  vms.push(vm);
		}
	  }
	}

	System.log("Checking to see if " + VMName + " is found.");

	if ( vms.length == 0 ) {
	  throw "vm not found";
	}
	if ( vms.length != 1 ) {
	  throw "multiple vms found with name " + VMName;
	}
	var vm = vms[0];
  
  This is fairly simple, but it was a good exercise for figuring out how to use JavaScript in the context of the vRO SDK/API.  (An excellent resource that anyone intending to write Scriptable Tasks in vRO should be familar with is www.vroapi.com/)

This script snippet starts with a very simple method call:

	vCenters = VcPlugin.allSdkConnections;


All this snippet does is create an Sdk connection to all of the vCenters associated with the vRO device you're connected to.  Once I have connected to each of these, I'm able to loop through each vCenter, and get a listing of VMs.

	var allVms = vCenter.getAllVirtualMachines();

I then loop through each of those VMs, and check the name property to see if it matches the name I provided.  If it matches the provided VMName, it pushes the vm into an array (vms), and then continues searching until it has exhausted all vms in all vcenters.  The reason I don't just stop when I find the VM, is that it is entirely possible that multiple VMs exist (especially on multiple vCenters) where they both have the same name.  It's not good, but it is possible.  So at the end of the script, I check the length of my vms array, and if it is 0 or greater than 1, I exit with an error; and so this script only returns successfully if only one VM is found.
