# Ssd-Function

this is an example of how to get an nt/ze function in kernel mode without opening the dll ntdll.dll and mapping it

	void* GetSsdFunction(const char* Name)
	{
	    void* Function = 0;

	    auto MiState = calc_offset<uint64_t>(ResolveRelativeAddress(FindPatternImage(ntosBase,
		    "4C 8D 1D ? ? ? ? 49 BE ? ? ? ? ? ? ? ? F7 84 24 ? ? ? ? ? ? ? ? 49 B9"), 3)); //_MI_SYSTEM_INFORMATION

	    if (!MiState) return 0;

	    auto SystemDllBase = *(void**)(MiState + 0x1550);

	    auto ExportAddress = reinterpret_cast<unsigned char*>(GetSystemExport(SystemDllBase, Name));
	    if (!ExportAddress) return 0;

	    int ssdt_offset = -1;
	    for (auto i = 0; i < 10; i++)
	    {
		    if (ExportAddress[i] == 0xC2 || ExportAddress[i] == 0xC3)
			    break;

		    if (ExportAddress[i] == 0xB8) {
			    ssdt_offset = *reinterpret_cast<int*>(ExportAddress + i + 1);
			    break;
		    }
	    }

	    if (ssdt_offset == -1) return 0;

	    return reinterpret_cast<unsigned char*>(KeServiceDescriptorTable->ServiceTableBase) + (reinterpret_cast<long*>(KeServiceDescriptorTable->ServiceTableBase)[ssdt_offset] >> 4);
	}

 ### Example usage
 auto NtSuspendThread = GetSsdFunction("NtSuspendThread");

more info:
https://www.vergiliusproject.com/kernels/x64/Windows%2010%20|%202016/2004%2020H1%20(May%202020%20Update)/_MI_SYSTEM_INFORMATION
