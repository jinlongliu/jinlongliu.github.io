---
layout: post
title: "Fabric-java-sdk示例"
description: ""
category: Blockchain 
tags: [区块链,Hyperledger, Fabric, Chaincode]
---
{% include JB/setup %}
**<font color="red">版权声明：本文为博主原创文章，转载注明“龙棠博客”字样和原文链接。</font>**

### Fabric-java-sdk 示例

{% highlight bash %}
package org.playchain.demo3.service;

import org.hyperledger.fabric.sdk.*;
import org.hyperledger.fabric.sdk.security.CryptoSuite;
import org.playchain.demo3.controller.SampleOrg;
import org.playchain.demo3.controller.SampleStore;
import org.playchain.demo3.controller.SampleUser;
import org.playchain.demo3.model.House;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.io.File;
import java.net.URL;
import java.nio.charset.StandardCharsets;
import java.util.*;

import static org.apache.commons.codec.CharEncoding.UTF_8;

/**
 * Created by Liu on 2017/9/21.
 */
@Service
public class ChaincodeService {
    private static final Logger logger = LoggerFactory.getLogger(ChaincodeService.class);

    private HFClient hfClient;
    private ChaincodeID chaincodeID;
    private Channel channel;
    private Collection<ProposalResponse> successful;

    @Autowired
    protected void init() {
        logger.info("Init chaincode service");
        successful = new LinkedList<>();
        File sampleStoreFile = new File("D:/" + "/HFCSampletest.properties");
        if (sampleStoreFile.exists()) { //For testing start fresh
            sampleStoreFile.delete();
        }

        final SampleStore sampleStore = new SampleStore(sampleStoreFile);
        SampleOrg sampleOrg = new SampleOrg("peerOrg1", "Org1MSP");
        try {
            ClassLoader classLoader = getClass().getClassLoader();
            URL channelUrl = classLoader.getResource("channel/");
            URL keystoreUrl = classLoader.getResource("channel/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/sk");
            URL signcertsUrl = classLoader.getResource("channel/crypto-config/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem");
            SampleUser peerOrgAdmin = sampleStore.getMember("Org1MSP" + "Admin", "Org1MSP", sampleOrg.getMSPID(),
                    new File(keystoreUrl.getFile()),
                    new File(signcertsUrl.getFile()));
//                    Paths.get(keystoreUrl.getFile()).toFile(),
//                    Paths.get(signcertsUrl.getPath()).toFile());

            sampleOrg.setPeerAdmin(peerOrgAdmin);
            hfClient= HFClient.createNewInstance();
            hfClient.setCryptoSuite(CryptoSuite.Factory.getCryptoSuite());

            hfClient.setUserContext(sampleOrg.getPeerAdmin());

            chaincodeID = ChaincodeID.newBuilder().setName("playchain_chaincode")
                    .setVersion("1")
                    .setPath("github.com/playchain").build();

            ChannelConfiguration channelConfiguration = new ChannelConfiguration(new File(channelUrl.getPath() +"/foo.tx"));


            URL servercrtUrl = classLoader.getResource("channel/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt");
//            File cert = Paths.get(channelUrl.getPath() +"/crypto-config/ordererOrganizations/example.com/orderers/orderer.example.com/tls/server.crt").toFile();

            File cert = new File(servercrtUrl.getFile());

            Properties ret = new Properties();
            ret.setProperty("pemFile", cert.getAbsolutePath());
            ret.setProperty("trustServerCertificate", "true"); //testing environment only NOT FOR PRODUCTION!
            ret.setProperty("hostnameOverride", "orderer.example.com");
            ret.setProperty("sslProvider", "openSSL");
            ret.setProperty("negotiationType", "TLS");

            Orderer newOrder = hfClient.newOrderer("orderer.example.com", "grpc://127.0.0.1:7050", ret);

            channel = hfClient.newChannel("foo");
            channel.addOrderer(newOrder);


            URL servercrtUrl0 = classLoader.getResource("channel/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt");
//            File cert0 = Paths.get(channelUrl.getPath() +"/crypto-config/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/server.crt").toFile();
            File cert0 = new File(servercrtUrl0.getFile());
            Properties ret0 = new Properties();
            ret0.setProperty("pemFile", cert.getAbsolutePath());
            ret0.setProperty("trustServerCertificate", "true"); //testing environment only NOT FOR PRODUCTION!
            ret0.setProperty("hostnameOverride", "peer0.org1.example.com");
            ret0.setProperty("sslProvider", "openSSL");
            ret0.setProperty("negotiationType", "TLS");


            Peer peer0 = hfClient.newPeer("peer0.org1.example.com", "grpc://localhost:7051", ret0);

            channel.addPeer(peer0);
//            newChannel.addPeer(peer1);
            channel.initialize();

        } catch (Exception e) {
            logger.info(e.getMessage());
        }
    }

    public void add(House house, SampleUser user) {
        try {

            hfClient.setUserContext(user);
            TransactionProposalRequest transactionProposalRequest = hfClient.newTransactionProposalRequest();
            transactionProposalRequest.setChaincodeID(chaincodeID);
            transactionProposalRequest.setFcn("registerHouse");
//            transactionProposalRequest.setProposalWaitTime(testConfig.getProposalWaitTime());
//            transactionProposalRequest.setArgs(new String[] {"move", "a", "b", "100"});
            String argS = house.getHouseNo() + "," + house.getHouseCertNo() + "," + house.getHouseLocation() + ","
                    + "owner" + "," + "user" + "," + "sale" + "," + "1000";
            String[] args = argS.split(",");
//            transactionProposalRequest.setArgs(new String[] {name,"blue","35","tom"});
            transactionProposalRequest.setArgs(args);

            Map<String, byte[]> tm2 = new HashMap<>();
            tm2.put("HyperLedgerFabric", "TransactionProposalRequest:JavaSDK".getBytes(StandardCharsets.UTF_8)); //Just some extra junk in transient map
            tm2.put("method", "TransactionProposalRequest".getBytes(StandardCharsets.UTF_8)); // ditto
            tm2.put("result", ":)".getBytes(StandardCharsets.UTF_8));  // This should be returned see chaincode why.
//            tm2.put(EXPECTED_EVENT_NAME, EXPECTED_EVENT_DATA);  //This should trigger an event see chaincode why.

            transactionProposalRequest.setTransientMap(tm2);


            Collection<ProposalResponse> transactionPropResp = channel.sendTransactionProposal(transactionProposalRequest, channel.getPeers());
            successful.clear();
            for (ProposalResponse response : transactionPropResp) {
                if (response.getStatus() == ProposalResponse.Status.SUCCESS) {
                    logger.info(response.toString());
                    successful.add(response);
                    logger.info(response.toString());
//                    return response.toString();
                } else {
                    logger.info("error");
                }
            }

            Collection<Set<ProposalResponse>> proposalConsistencySets = SDKUtils.getProposalConsistencySets(transactionPropResp);
            if (proposalConsistencySets.size() != 1) {
                logger.info("error");
            }

            channel.sendTransaction(successful);

        } catch (Exception e) {
            logger.info(e.toString());
        }
    }

    public void query(String name) {
        try {
            QueryByChaincodeRequest queryByChaincodeRequest = hfClient.newQueryProposalRequest();
//            queryByChaincodeRequest.setArgs(new String[] {"query", "b"});
            queryByChaincodeRequest.setArgs(new String[] {name});

            queryByChaincodeRequest.setFcn("queryHouse");
            queryByChaincodeRequest.setChaincodeID(chaincodeID);

            Map<String, byte[]> tm2 = new HashMap<>();
            tm2.put("HyperLedgerFabric", "QueryByChaincodeRequest:JavaSDK".getBytes(UTF_8));
            tm2.put("method", "QueryByChaincodeRequest".getBytes(UTF_8));
            queryByChaincodeRequest.setTransientMap(tm2);


            Collection<ProposalResponse> queryProposals = channel.queryByChaincode(queryByChaincodeRequest, channel.getPeers());

            for (ProposalResponse proposalResponse : queryProposals) {
                if (!proposalResponse.isVerified() || proposalResponse.getStatus() != ProposalResponse.Status.SUCCESS) {
                    logger.info("xxxx");
                } else {
                    String payload = proposalResponse.getProposalResponse().getResponse().getPayload().toStringUtf8();
                    logger.info(payload);
//                    return  payload;
                }
            }
        } catch (Exception e) {
            logger.info(e.toString());
        }
    }

}

{% endhighlight %}


{% include JB/setup %}