```
void Document__init( Document *D  ) {
	
	D->C.index = std::this_thread::get_id();
	D->V.Window->Init();
	
	Host::Transfer Tx = make_shared<Host::Transfer_Object>();
	Tx->URL = D->Manifest; Hosts.Request(Tx);
	D->Parse(Tx->Data);
	
	while (true) {
		std::unique_lock<std::mutex> lock(*D->C.mutex);
		(*D->C.ready).wait(lock);
		
		while(D->C.Queue.size()) {
			constructor::Event Ev = D->C.Queue.front();
									D->C.Queue.pop_front();

			if(Ev.Request != NULL) {
				D->C.Engine->Receive(Ev.Request);
			} else
			if(Ev.Element.s["type"] != "") {
				D->C.Engine->Dispatch(Ev.Element);
			}
			//-------------------------
			if (D->Manifest == "") {
				delete D;
				return;
			}
		}
	}
	
};
Document::Document( string Manifest ) {	
	Documents.push_back(this);
	this->Manifest = Manifest;
	
	this->M.D = 
	this->V.D = 
	this->C.D = this;
	
	this->M.Root = Element("root");
	
	this->V.Window = new System::Window(this);
	
	this->C.mutex = new std::mutex();
	this->C.ready = new std::condition_variable();
	this->C.Thread = std::thread(Document__init, this);
	this->C.Thread.detach();
};
```