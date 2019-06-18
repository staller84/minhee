pragma solidity ^0.4.18 ;

contract TLT_Token {	
    address owner ;
//// TLT Token 생성 
    string  constant name = "TLT Token" ;
    string  constant symbol = "TLT" ;
    uint8   constant decimals = 0 ; 
    
    mapping ( address => uint ) public balanceOf ; 
    
    event Transfer( address from, address to, uint value ) ;

    function transfer( address _to, uint _value) public {
        address _from = msg.sender ;
        require( _to != address(0) ) ;
        require( balanceOf[_from] >= _value ) ;
        balanceOf[_from] -= _value ;
        balanceOf[_to] += _value ; 
    }
    
    function TLT_token() public {
        balanceOf[msg.sender] = 1*10**15 ; 
    }
    
}

contract voteContract {	
	struct cand_info {  //구직자 구조체 - 구직자 정보관리용 
        string[ ] 	    attrbute ;   //역량 
		uint8[ ]    	true_cnt ; 
		uint8[ ]	    false_cnt ;
		uint8[ ]	    dntknow_cnt ;  
	}

	struct voter_info { //투표자 구조체 
 		string[ ] 	    cand ;  
        string[ ] 	    attrbute2 ;   //역량		
		bool[ ]	        true_ ; 
		bool[ ]	        false_ ;
		bool[ ]	        dntknow_ ; 
		address[ ]      voters ; 
	}

    mapping( string   => cand_info  ) cand_info2 ;  
    mapping( string   => mapping( string => cand_info )  ) cand_info3 ;      
    
    mapping( address  => voter_info ) voter_info2 ; 
    mapping( address  => mapping( string  => voter_info) ) voter_info3 ; 
    
    mapping( string  => mapping( string  => voter_info) ) voter_info4 ;     
	mapping( string   => uint   ) 	candidates ;  	    // 득표수를 저장 
	mapping( uint8 	  => string ) 	candidatelist ;  	// 구직자 리스트 
    mapping( string => uint8 ) 	    numberofattrs ; 	// 총 역량 항목수  

	uint8    	numberofCandidates=0 ;  // 총 구직자 수 
	address 	Owner ; 
	address[]   Final_list ; 
	
	uint8       total_cnt ; 

	function voteContract() public { 	
		Owner = msg.sender ; 
  	}
	modifier ownerOnly {
		require( msg.sender == Owner ) ;
		_ ;
	}

	// 구직자 & 구직자 역량 항목 추가 
	function addCandattr( string cand2, string attr ) ownerOnly public {
		bool add  = true ; 
        bool add2 = true ; 
    
		for ( uint8 i=0 ; i < numberofCandidates ; i++ ) {
			if ( sha256( candidatelist[i] ) == sha256( cand2 ) ) {
				add = false ;  //이미 있는 구직자의 경우는 추가 하지 않음    
				break ; 
			}
		}
	
		if( add ) 
		{	 //기존에 없는 경우는 구직자 리스트에 추가 
			candidatelist[ numberofCandidates ] = cand2 ;
            numberofattrs[ cand2 ] = 0 ; 
			numberofCandidates ++ ; 
  		}
  		
        //구직자 역량 추가 부분 
     	for ( i=0 ; i < numberofattrs[ cand2 ] ; i++ ) {
             if( sha256( cand_info2[ cand2 ].attrbute[i] ) == sha256( attr ) ) {
                 add2=false ; 
                 break ; 
             }
        }
        
        if( add2 )
        {
            cand_info2[ cand2 ].attrbute[ numberofattrs[cand2] ] = attr ;       
            numberofattrs[cand2] ++ ;
        }

 	}	
 
	// 구직자 역량에 투표를 하는 함수 
	function vote(string cand3, string attr2, bool true_2, bool false_2, bool dntknow_2 ) public {
	//	require( msg.sender != Owner ) ; 
		total_cnt ++ ; 
		for ( uint8 i=0 ; i < numberofCandidates ; i++ ) {
			if( sha3( candidatelist[i] ) == sha3( cand3 ) ) {
			   voter_info2[msg.sender].cand[i]      = cand3 ;
			    					    
				for ( uint8 j=0 ; j < numberofattrs[cand3] ; j++ ) {
					if( sha3( cand_info2[cand3].attrbute[j] ) == sha3( attr2 ) ) {
					    
					    voter_info3[msg.sender][cand3].attrbute2[j] = attr2 ;					    
					    voter_info3[msg.sender][cand3].true_[j]     = true_2 ;
					    voter_info3[msg.sender][cand3].false_[j]    = false_2 ;
					    voter_info3[msg.sender][cand3].dntknow_[j]  = dntknow_2 ;					    
		
						if ( voter_info3[msg.sender][cand3].true_[j] == true ) {
							cand_info3[cand3][attr2].true_cnt[j] ++ ;
						}
						else if ( voter_info3[msg.sender][cand3].false_[j] == true ) {
							cand_info3[cand3][attr2].false_cnt[j] ++ ;
						}
						else if ( voter_info3[msg.sender][cand3].dntknow_[j] == true ) {
							cand_info3[cand3][attr2].dntknow_cnt[j] ++ ;
						}	
						else {} 
						
					    voter_info3[msg.sender][cand3].voters[j]  = msg.sender ;						
					}
				}
			}
		}
	}

    function vote_result_list() public returns(string) {
		for ( uint8 i=0 ; i < numberofCandidates ; i++ ) {
            for ( uint8 j=0 ; j < numberofattrs[ candidatelist[i] ] ; j++ ) {
                for( uint8 k=0 ; k < total_cnt ; k++ ) {
                        if ( cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].true_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].false_cnt[j] &&    
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].true_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].dntknow_cnt[j] &&
                             voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].true_[k] == true ) 
                             {
                        Final_list[k] = voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].voters[k] ; 
                             }
                    else if ( cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].false_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].true_cnt[j] &&    
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].false_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].dntknow_cnt[j] &&
                             voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].false_[k] == true ) 
                             {
                        Final_list[k] = voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].voters[k] ; 
                             }                             
                    else if ( cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].dntknow_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].true_cnt[j] &&    
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].dntknow_cnt[j] >= 
                             cand_info3[ candidatelist[i] ][ cand_info2[candidatelist[i]].attrbute[j] ].false_cnt[j] &&
                             voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].dntknow_[k] == true ) 
                             {
                        Final_list[k] = voter_info4[candidatelist[i]][cand_info2[candidatelist[i]].attrbute[j]].voters[k] ; 
                             }       
                    else { }
                }  
            }
		}
    }

//// 최종 대상자에 TLT Token 송금 
    function vote_result_list_transfer() public payable {
        uint balance=3 ;
            for( uint8 k=0 ; k < total_cnt ; k++ ) {
                Final_list[k].transfer(address(this).balance) ;
            }
    }    
    
}