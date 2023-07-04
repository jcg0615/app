import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';

class UserWidget extends StatefulWidget {
  @override
  _UserWidgetState createState() => _UserWidgetState();
}

class _UserWidgetState extends State<UserWidget> {
  int mValue = 0;
  int couponValue = 0;

  @override
  void initState() {
    super.initState();
    fetchUserData();
  }

  Future<void> fetchUserData() async {
    try {
      DocumentSnapshot snapshot = await FirebaseFirestore.instance.collection('test').doc('user').get();

      if (snapshot.exists && snapshot.data().containsKey('M') && snapshot.data().containsKey('coupon')) {
        setState(() {
          mValue = snapshot.data()['M'];
          couponValue = snapshot.data()['coupon'];
        });
      }
    } catch (e) {
      print('사용자 데이터를 가져오는 동안 오류가 발생했습니다: $e');
    }
  }

  Future<void> updateUserFields() async {
    int updatedMValue = mValue;
    int updatedCouponValue = couponValue;

    if (mValue >= 5000) {
      updatedMValue -= 5000;
      updatedCouponValue++;
    } else {
      showDialog(
        context: context,
        builder: (BuildContext context) {
          return AlertDialog(
            title: Text('알림'),
            content: Text('최소 마일리지를 달성하지 못했습니다.'),
            actions: [
              TextButton(
                onPressed: () {
                  Navigator.pop(context);
                },
                child: Text('확인'),
              ),
            ],
          );
        },
      );
      return;
    }

    try {
      await FirebaseFirestore.instance.collection('test').doc('user').update({
        'M': updatedMValue,
        'coupon': updatedCouponValue,
      });

      setState(() {
        mValue = updatedMValue;
        couponValue = updatedCouponValue;
      });
    } catch (e) {
      print('사용자 데이터를 업데이트하는 동안 오류가 발생했습니다: $e');
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('User Widget'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Container(
              width: 200,
              height: 200,
              decoration: BoxDecoration(
                border: Border.all(color: Colors.black),
                color: Colors.white,
              ),
              child: Center(
                child: Text(
                  'M Value: $mValue',
                  style: TextStyle(fontSize: 24),
                ),
              ),
            ),
            SizedBox(height: 16),
            Text(
              'Coupon: $couponValue',
              style: TextStyle(fontSize: 24),
            ),
            SizedBox(height: 16),
            ElevatedButton(
              onPressed: () {
                updateUserFields();
              },
              child: Text('사용'),
            ),
          ],
        ),
      ),
    );
  }
}

void main() {
  runApp(MaterialApp(
    home:
